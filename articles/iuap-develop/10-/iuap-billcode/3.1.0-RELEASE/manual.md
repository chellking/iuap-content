# 编码规则组件概述 #

## 业务需求 ##

在业务系统中，业务对象通常会有一个**业务对象编号**，业务对象编号作为业务对象具有业务意义的标识，一般由时间，序列号，常量等子段按照一定的规则拼接而成，这个规则我们称为'**编码规则**'，各个子段我们称之为'编码元素'。考虑如下的业务对象编号：'TT-20150615-0001',编码规则为:（常量'TT-'+yyyyMMdd类型的时间'20150615'+常量'-'+序列号'0001'),如果时间变为'20150616',序列号重新从'0001'开始流水，那么我们称此时间编码元素是'流水依据元素',所有的流水依据元素合起来叫做本编码规则的'流水依据'，同一个编码规则，可以有很多个流水依据！  


##解决方案##

目前比较少编码规则的开源组件，网上能找到的也只是一些简单的实现类或者设计思路，要么将规则写死在代码中，缺乏通用性和灵活性，要么缺少全面的考虑，本组件提供了一种通用的编码规则解决方案。
本组件的核心是根据编码规则和一些必要信息，生成业务对象编码。组件需要的编码规则可以存储在文件中，甚至是手工构造，或者存储在数据库中。本组件提供了存储在数据库中一种实现，提供了基本的增加，删除，修改编码规则的服务方法，用户可以在此调用此服务，开发出自己的编码规则设计工具，也可以直接设计符合自己要求的编码规则数据库结构和服务，编码规则VO符合组件的模型即可（实现组件的规则接口）。  

另外，组件提供了数据库行锁和zookeeper分布式锁的支持，以应对大并发的情况下可能出现的编码重复问题。

## 功能说明 ##

1.	根据编码规则和一些必要信息，生成业务对象编码；
3.	支持灵活的规则定义，包括业务数据属性、日期、时间、常量、流水等；
4.	支持最大号定义和归零规则定义；
5.	支持扩展机制，支持业务属性获取，支持自定义序列号生成算法、支持系统时间的扩展、支持自定义随机码算法
6.	支持批量取号、退号等操作；
7.	支持编码补号
7.	支持高并发场景下不重号、丢号；


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：
```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-billcode</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>
```
${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 流程说明 ##
编码规则分为前编码与后编码，使用过程如下：

1.	界面新增时，先调用获取编码规则上下文接口，如果存在编码规则，根据上下文控制编码字段的可编辑性等。
2.  如果上下文为前编码规则时，调用前编码预取单据号接口获取编码，
3.  在保存服务中调用提交预取的前编码单据编码接口，然后保存数据。
4.  如果界面直接取消保存时，调用回滚预取单据号口。
3.	如果上下文为前编码规则时，界面编码字段默认为空。
4.	在保存服务中，调用获取编码规则API，获取编码规则填充到数据对象中保存。
5.	删除数据时，调用删除单据时回退单据号接口进行退号操作。

# 使用说明 #

## 开发步骤 ##


可参见具体的示例工程：


### 执行数据库脚本 ###

依次执行examples项目下sql目录中的dll.sql、index.sql、dml.sql建立数据库并初始化数据。


### spring集成 ###

** 1. 在Spring的配置文件中添加如下配置（或者是将以下内容放在一个单独的配置文件中，在工程启动时加载此文件）：**

```
  	<?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
      xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
      xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd"
      default-lazy-init="true">

      <description>Spring公共配置</description>

      <!-- 使用annotation 自动注册bean, 并保证@Required、@Autowired的属性被注入，排除@Controller注解 -->
      <context:annotation-config />
      <context:component-scan base-package="com.yonyou.uap.billcode">
        <context:exclude-filter type="annotation"
          expression="org.springframework.stereotype.Controller" />
      </context:component-scan>

      <!-- Mybatis配置 -->
      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" value="classpath:mybatis-mappers/*.xml" />
      </bean>

      <!-- 扫描DAO接口 -->
      <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.yonyou.uap.billcode.repository" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
      </bean>

      <!-- 事物管理 -->
      <!-- 开启事物管理注解 -->
      <!-- <tx:annotation-driven transaction-manager="transactionManager" -->
      <!-- proxy-target-class="true" /> -->
      <!-- <bean id="transactionManager" -->
      <!-- class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> -->
      <!-- <property name="dataSource" ref="dataSource" /> -->
      <!-- </bean> -->

      <!-- Spring jdbcTemplate -->
      <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate"
        abstract="false" lazy-init="false" autowire="default">
        <property name="dataSource">
          <ref bean="dataSource" />
        </property>
      </bean>

      <!-- 注入MySQL数据库行锁 -->
      <bean id="mySqlRowLock" class="com.yonyou.uap.billcode.lock.MySqlRowLock" />

      <!-- 注入zookeeper分布式锁 -->
    <!--  <bean id="zkLock" class="com.yonyou.uap.billcode.lock.ZKLock" /> -->

      <!-- 读取zookeeper分布式锁属性配置 -->
    <!--  <bean id="billcode-propertyConfigurer" -->
    <!--    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"> -->
    <!--    <property name="locations"> -->
    <!--      <list> -->
    <!--        <value>classpath:application.properties</value> -->
    <!--      </list> -->
    <!--    </property> -->
    <!--    <property name="systemPropertiesMode" -->
    <!--      value="#{T(org.springframework.beans.factory.config.PropertyPlaceholderConfigurer).SYSTEM_PROPERTIES_MODE_OVERRIDE}" /> -->
    <!--  </bean> -->

      <!-- zookeeper分布式锁配置加载 -->
    <!--  <bean id="zkPoolConfig" class="org.apache.commons.pool2.impl.GenericObjectPoolConfig"> -->
    <!--    <property name="maxTotal" value="100" /> -->
    <!--    <property name="maxIdle" value="100" /> -->
    <!--    <property name="maxWaitMillis" value="60000" /> -->

    <!--    <!--timeBetweenEvictionRunsMillis毫秒检查一次连接池中空闲的连接,把空闲时间超过minEvictableIdleTimeMillis毫秒的连接断开,直到连接池中的连接数到minIdle为止 -->
    <!--    <property name="timeBetweenEvictionRunsMillis" value="60000" /> -->
    <!--    <property name="numTestsPerEvictionRun" value="-1" /> -->
    <!--    <property name="minEvictableIdleTimeMillis" value="30000" /> -->
    <!--  </bean> -->
    </beans>
```

** 2. 如果要使用分布式锁的机制，需要将spring配置文件中的zookeeper相关的注释放开，将mySqlRowLock的注释掉。同时配置application.properties（也可以为自己指定的属性文件，只要注入到spring中即可）内容如下：**

```

#配置锁组件连接zookeeper时候使用的连接类型，single(单机)、cluster（集群）
zklock.connection.type=single
zklock.url.single=20.12.6.126:2181
zklock.url.cluster=172.20.12.20:2180,172.20.12.20:2181,172.20.12.20:2182

```
** 3. web.xml 配置启动监听初始化分布式锁的连接池（使用分布式锁时） **


代码实现
```
public class CustomServletContextListener implements ServletContextListener {

    public void contextDestroyed(ServletContextEvent sce) {
    }

	public void contextInitialized(ServletContextEvent event) {
		WebApplicationContext wac = WebApplicationContextUtils.getWebApplicationContext(event.getServletContext());
		ContextHolder.setContext(wac);
		GenericObjectPoolConfig config = (GenericObjectPoolConfig)ContextHolder.getContext().getBean("zkPoolConfig");
		ZkPool.initPool(config);
	}

}

```
web.xml中配置如下，

```

<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	<listener-class>com.yonyou.uap.billcode.listener.CustomServletContextListener</listener-class>
</listener>

```
** 注:此处主要为了初始化分布式锁，只要保证应用启动时执行ZkPool.initPool(config)即可**


### 使用API接口完成请求 ###

按照API接口 要求，调用对应API，管理编码规则或者生成编码

### 扩展机制 ###

编码规则组件支持对编码规则元素的自定义替换，通过上下文配置文件来实现自定义的编码规则组件设置（位于jar包'com/yonyou/uap/billcode/BillCodeEngineContext.xml'），如下

![](image/image001.jpg)    

支持以下几种扩展方式：  

** 一、自定义编码元素处理器 **  
1. 复制本jar包中  `'com/yonyou/uap/billcode/BillCodeEngineContext.xml'到任意'类路径''A'`
2. 实现编码元素处理器接口：  `'com.yonyou.uap.billcode.elemproc.itf.IElemProcessor'`  
3. 在编码规则引擎上下文配置文件（复制的`'BillCodeEngineContext.xml'`）中`'添加'`或`'替换'`元素处理器

** 二、自定义序列号编码元素的序列号生成方式 **  
1.同上  
2.实现编码元素处理器接口：      `'com.yonyou.uap.billcode.sngenerator.ISNGenerator'`  
3.在编码规则引擎上下文配置文件（复制的`'BillCodeEngineContext.xml'`）中`'添加'`或`'替换'`序列号生成器

** 三、自定义系统时间编码元素的时间获取方式 **  
1.同上  
2.实现编码元素处理器接口：      `'com.yonyou.uap.billcode.sysdate.ISysDateProvider'`  
3.在编码规则引擎上下文配置文件（复制的`'BillCodeEngineContext.xml'`）中`'替换'`时间获取方式

** 四、自定义唯一性随机编码生成类 ** （不需要业务对象编号有业务意义时，此类提供唯一性的无业务意义的编码）  
1.同上  
2.实现编码元素处理器接口：  `'com.yonyou.uap.billcode.randomcode.IRandomCodeGenerator'`  
3.在编码规则引擎上下文配置文件（复制的`'BillCodeEngineContext.xml'`）中`'替换'`随机编码生成类

注意：自定义编码规则组件的上下文，需要在程序启动时调用`com.yonyou.uap.billcode.BillCodeEngineContext.getInstance('A')`将引擎上下文A载入内存（否则将走默认的的上下文配置` 'com/yonyou/uap/billcode/BillCodeEngineContext.xml'`）


## 编码规则对象注册说明 ##
### 编码规则

详见com.yonyou.uap.billcode.model.IBillCodeBaseVO

<table>
  <tr>
    <th>字段名</th>
    <th>名称</th>
	<th>说明</th>
  </tr>
  <tr>
    <td>pk_billcodebase</td>
    <td>主键</td>
	<td></td>
  </tr>
  <tr>
    <td>rulecode</td>
    <td>规则编码</td>
	<td></td>
  </tr>
  <tr>
    <td>rulecode</td>
    <td>规则编码</td>
	<td></td>
  </tr>
  <tr>
    <td>rulename</td>
    <td>规则名称</td>
	<td></td>
  </tr>
  <tr>
    <td>codemode</td>
    <td>编码方式</td>
		<td>前编码或后编码，具体见IBillCodeBaseVO中常量</td>
  </tr>
  <tr>
    <td>iseditable</td>
    <td>生成的编码是否可以编辑</td>
		<td>true/false</td>
  </tr>
  <tr>
    <td>isautofill</td>
    <td>编码是否保证连</td>
		<td>true/false，为true时，系统会重新使用被删除或取消的流水号，否则取消或删除的流水号不会在使用</td>
  </tr>
  <tr>
    <td>isBolGetRandomCode</td>
    <td>是否获取随机码</td>
		<td>true/false，设置单据号是否以随机数的方式生成还是以编码元素定义的方式生成；两者只能选择一种，后续的编码元素要根据选则设置</td>
  </tr>
  <tr>
    <td>isBolSNAppendZero</td>
    <td>序列号是否补零</td>
		<td>true/false，不补0，序列号4位，1显示为1，否则，为0001，用于流水号生成算法</td>
  </tr>
</table>

** 编码方式: ** IBillCodeBaseVO.STYLE_XXX

<table>
  <tr>
    <th>常量</th>
	  <th>常量值</th>
    <th>描述</th>
		<th>具体说明</th>
  </tr>
  <tr>
    <td>STYLE_PRE</td>
    <td>pre</td>
    <td>前编码</td>
		<td>即界面新增时获取编码，分两步调用，先预取编码设置到编码字段上，然后单据保存时提交编码，否则回滚编码</td>
  </tr>
  <tr>
    <td>STYLE_AFT</td>
    <td>after</td>
    <td>后编码</td>
		<td>即数据后台保存时获取编码，新增时编码字段为空，在单据保存时，调用获取编码</td>
  </tr>
</table>

### 编码规则元素

详见com.yonyou.uap.billcode.model.IBillCodeElemVO

<table>
  <tr>
    <th>字段名</th>
    <th>名称</th>
	<th>说明</th>
  </tr>
  <tr>
    <td>pk_billcodeelem</td>
    <td>主键</td>
	<td></td>
  </tr>
  <tr>
    <td>pk_billcodebase</td>
    <td>编码规则元素主键</td>
	<td></td>
  </tr>
  <tr>
    <td>elemtype</td>
    <td>元素类型</td>
		<td>见下面说明</td>
  </tr>
  <tr>
    <td>elemvalue</td>
	    <td>元素值</td>
			<td>不同元素类型的编码元素意义不同：具体参考见下面说明
			<br>TYPE_SN-流水号 			：流水号生成器的类型 具体值为参考下文
		  <br>TYPE_CONST-常量     			：常量值
		  <br>TYPE_SYSTIME-系统时间		：空
		  <br>TYPE_RANDOM-随机码			：空
		  <br>TYPE_BILLBUSITIMEATTR-单据业务时间	：扩展人员决定是否使用
		  <br>TYPE_BILLSTRINGATTR-单据字符串属性     ：扩展人员决定是否使用
		  <br>TYPE_BILLENTITYATTR-单据参照属性	：扩展人员决定是否使用
		</td>
  </tr>
  <tr>
    <td>elemlenth</td>
    <td>编码元素的长度</td>
	<td></td>
  </tr>
  <tr>
    <td>isrefer</td>
    <td>元素流水依据</td>
		<td>见下面说明</td>
  </tr>
  <tr>
    <td>eorder</td>
    <td>编码元素的排序</td>
		<td>最终的单据编码按此顺序拼接而成</td>
  </tr>
  <tr>
    <td>fillstyle</td>
    <td>补位方式</td>
		<td>仅“单据字符串属性”，“单据参照属性”时使用，其它元素类型请选择“不补位”具体值见下面说明</td>
  </tr>
  <tr>
    <td>fillsign</td>
    <td>补位符号</td>
		<td>补位方式不为“不补位”时使用，补位时增补的字符，例如补位符号为“@”，字段值为001，长度为5时，补位方式为右补位时，生成元素值为“001@@”</td>
  </tr>
  <tr>
    <td>datedisplayformat</td>
    <td>日期格式化方式</td>
	<td>只有元素是日期类型时才会用到，最终日期的格式，具体值见下面说明</td>
  </tr>
</table>

** 元素类型: ** IBillCodeElemVO.TYPE_XXX

<table>
  <tr>
    <th>常量</th>
	  <th>常量值</th>
    <th>描述</th>
		<th>具体说明</th>
  </tr>
  <tr>
    <td>TYPE_SN</td>
    <td>0</td>
    <td>流水号</td>
		<td>系统根据流水号生成策略生成，一般从1开始递增</td>
  </tr>
  <tr>
    <td>TYPE_CONST</td>
    <td>1</td>
    <td>常量</td>
		<td>指定长度的字符，例如“AAA”</td>
  </tr>
  <tr>
    <td>TYPE_SYSTIME</td>
    <td>2</td>
    <td>系统时间</td>
		<td>生成编码时的系统时间，最后生成的编码内容，根据编码元素中指定的日期格式化方式生成，例如指定为“yyyyMMdd”生成的格式为“20160721”</td>
  </tr>
  <tr>
    <td>TYPE_RANDOM</td>
    <td>3</td>
    <td>随机码</td>
		<td>系统根据编码元素长度自动生成的随机的字符串</td>
  </tr>
  <tr>
    <td>TYPE_BILLBUSITIMEATTR</td>
    <td>4</td>
    <td>单据业务时间</td>
		<td>从单据实体上获取单据的时间值，同系统时间根据指定的日期格式化方式生成，需要实现IBillVOFieldValueFetcher接口，具体见接口扩展说明</td>
  </tr>
  <tr>
    <td>TYPE_BILLSTRINGATTR</td>
    <td>5</td>
    <td>单据字符串属性</td>
		<td>单据的属性的值，需要实现IBillVOFieldValueFetcher接口，具体见接口扩展说明</td>
  </tr>
  <tr>
    <td>TYPE_BILLENTITYATTR</td>
    <td>6</td>
    <td>单据参照属性</td>
		<td>单据上的参照属性，返回需要实现IBillVOFieldValueFetcher接口，具体见接口扩展说明</td>
  </tr>
</table>

** 流水号生成器的类型: ** IBillCodeElemVO.TYPE_XXSN_GENETATOR_XXX

<table>
  <tr>
    <th>常量</th>
	  <th>常量值</th>
    <th>描述</th>
		<th>具体说明</th>
  </tr>
  <tr>
    <td>SN_GENETATOR_PUREDIGITAL</td>
    <td>0</td>
    <td>数字</td>
		<td></td>
  </tr>
  <tr>
    <td>SN_GENETATOR_LETTERDIGITAL</td>
    <td>1</td>
    <td>流水号生成规则为首位为字母，后位为数字，亦即流水号首位为26进制，其余位数为10进制</td>
		<td>对于流水号补位和流水号不补位的情况，假设前台设置的流水号位数为5位:
	  <br>1.流水号补位：具体流水号如下，第一个流水号 A0000， 第二个流水号 A0001，最后一个流水号 Z9999
	  <br>2.流水号不补位： 具体流水号如下： 第一个流水号A0， 第二个流水号 A1， 最后一个流水号 Z9999 特别的，假设前台设置的流水号位数为1位:流水号为A-Z。调用时第一个传入的sn转换为long后为0
		</td>
  </tr>
</table>

** 元素流水依据: ** IBillCodeElemVO.REF_XXX

<table>
  <tr>
    <th>常量</th>
	  <th>常量值</th>
    <th>描述</th>
		<th>具体说明</th>
  </tr>
  <tr>
    <td>REF_NOT</td>
    <td>0</td>
    <td>不作为流水依据</td>
		<td></td>
  </tr>
  <tr>
    <td>REF_YEAR</td>
    <td>1</td>
    <td>时间类型的流水依据---按年流水</td>
		<td>仅时间类型使用，即“系统时间”或“单据业务时间”</td>
  </tr>
  <tr>
    <td>REF_MON</td>
    <td>2</td>
    <td>时间类型的流水依据---按月流水</td>
		<td>仅时间类型使用，即“系统时间”或“单据业务时间”</td>
  </tr>
  <tr>
    <td>REF_DAY</td>
    <td>3</td>
    <td>时间类型的流水依据---按日流水</td>
		<td>仅时间类型使用，即“系统时间”或“单据业务时间”</td>
  </tr>
  <tr>
    <td>REF_REALVALUE</td>
    <td>4</td>
    <td>按实际值流水</td>
		<td>“单据字符串属性”或“单据参照属性”使用，为对应的单据字段值或参照对应实体的编码实际值进行流水
			<br>单据字符串属性：IBillVOFieldValueFetcher.getBillStrAttrVal(IBillCodeElemVO elem, Object bill)返回值进行流水
			<br>单据参照属性：IBillVOFieldValueFetcher.getBillEntityAttrVal(IBillCodeElemVO elem, Object bill)返回值进行流水
		</td>
  </tr>
  <tr>
    <td>REF_CUTEDVALUE</td>
    <td>5</td>
    <td>按设置的长度截取后的值流水</td>
		<td>同上，但根据上面实际值经截取或补位后的生成的编码段的值，进行流水</td>
  </tr>
  <tr>
    <td>REF_ENTITYKEY</td>
    <td>6</td>
    <td>按最终参照的编码实体的主键流水</td>
		<td>“单据参照属性”使用，根据参照的主键值流水，IBillVOFieldValueFetcher.getBillEntityAttrKey(IBillCodeElemVO elem, Object bill)</td>
  </tr>
</table>

** 补位方式: ** IBillCodeElemVO.FIllSTYLE_XXX

<table>
  <tr>
    <th>常量</th>
	  <th>常量值</th>
    <th>描述</th>
		<th>具体说明</th>
  </tr>
  <tr>
    <td>FIllSTYLE_NOT</td>
    <td>0</td>
    <td>不补位</td>
		<td></td>
  </tr>
  <tr>
    <td>FIllSTYLE_LEFT</td>
    <td>1</td>
    <td>左补位</td>
		<td></td>
  </tr>
  <tr>
    <td>FIllSTYLE_RIGHT</td>
    <td>2</td>
    <td>右补位</td>
		<td></td>
  </tr>
</table>

** 日期格式化方式: ** IBillCodeElemVO.DATEFORMAT_XX

<table>
  <tr>
    <th>常量</th>
	  <th>常量值</th>
    <th>描述</th>
		<th>具体说明</th>
  </tr>
  <tr>
    <td>DATEFORMAT_YY</td>
    <td>yy</td>
    <td>显示年份：2016/7/21 格式化为16</td>
		<td></td>
  </tr>
  <tr>
    <td>DATEFORMAT_YYYY</td>
    <td>yyyy</td>
    <td>左补位</td>
    <td>显示年份：2016/7/21 格式化为2016</td>
  </tr>
  <tr>
    <td>DATEFORMAT_YYMM</td>
    <td>yyMM</td>
    <td>右补位</td>
    <td>显示年月：2016/7/21 格式化为1607</td>
  </tr>
  <tr>
    <td>DATEFORMAT_YYYYMM</td>
    <td>yyyyMM</td>
    <td>右补位</td>
    <td>显示年月日：2016/7/21 格式化为201607</td>
  </tr>
  <tr>
    <td>DATEFORMAT_YYMMDD</td>
    <td>yyMMdd</td>
    <td>右补位</td>
    <td>显示年月：2016/7/21 格式化为160721</td>
  </tr>
  <tr>
    <td>DATEFORMAT_YYYYMMDD</td>
    <td>yyyyMMdd</td>
    <td>右补位</td>
    <td>显示年月日：2016/7/21 格式化为20160721</td>
  </tr>
</table>

## API接口 ##

### 公共参数说明

#### BillCodeElemInfo

**描述**  

一般为为空。用于临时指定一些参数，做为附加的编码规则字段使用。

**参数说明**

<table>
  <tr>
    <th>参数字段</th>
		<th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>elemResultValue</td>
		<td>String</td>
    <td>编码元素值，补充到生成编码的最后</td>
  </tr>
  <tr>
    <td>elemLength</td>
		<td>String</td>
    <td>编码元素值的长度</td>
  </tr>
</table>  

#### BillCodeBillVO

**描述**  

在编码元素包含“单据业务时间”、“单据字符串属性”、“单据参照属性”时使用，用于获取业务对象的属性值。

**参数说明**

<table>
  <tr>
    <th>参数字段</th>
		<th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>billVO</td>
		<td>Object</td>
    <td>业务对象，类型为具体业务系统的实体类型，IBillVOFieldValueFetcher接口中使用</td>
  </tr>
  <tr>
    <td>valueFetcher</td>
		<td>com.yonyou.uap.billcode.model.IBillVOFieldValueFetcher</td>
    <td>业务对象属性获取接口，接口说明如下</td>
  </tr>
</table>  

#### IBillVOFieldValueFetcher接口说明

**描述**  

用于根据业务对象获取相关的业务属性，生成编码使用

**方法**

**1. 获取业务时间：getBillBusiTimeAttrVal(IBillCodeElemVO elem, Object bill)**

返回Date类型，不需要处理格式化问题

**2. 获取单据的String属性值：getBillStrAttrVal(IBillCodeElemVO elem, Object bill)**

返回指定的单据字段的实际值，不需要处理截取或补位

**3. 获取单据参照属性的实际编码值：getBillEntityAttrVal(IBillCodeElemVO elem, Object bill)**

返回指定的单据参照字段中记录的主键对应的业务实体的编码，也可以是其他用来表示该实体的编码值，例如通过对象编码映射指定每一个对象对应的编码值。由业务系统决定实现方案。不需要处理截取或补位

**4. 获取单据参照属性的主键值：getBillEntityAttrKey(IBillCodeElemVO elem, Object bill)**

返回指定的单据参照字段中记录的主键值，不参与编码的生成，仅用于流水依据。




### 获取编码规则API ###

**描述**  

根据编码规则编码查询编码规则  

**请求方法**  

com.yonyou.uap.billcode.service.BillCodeRuleMgrService.getBillCodeRuleByRuleCode(String)  

**请求方式**  

服务调用  

**请求参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulecode</td>
    <td>True</td>
	<td>String</td>
    <td>  </td>
    <td>编码规则编号</td>
  </tr>
</table>  

**返回参数说明**  

`编码规则`  


### 删除编码规则的API ###

**描述**  

删除编码规则  

**请求方法**  

com.yonyou.uap.billcode.service.BillCodeRuleMgrService.delBillCodeRule(String)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulecode</td>
    <td>True</td>
	<td>String</td>
    <td>  </td>
    <td>编码规则编号</td>
  </tr>
</table>

**返回参数说明**  

无  


### 更新和保存编码规则的API ###

**描述**  

更新或保存编码规则  

**请求方法**  

com.yonyou.uap.billcode.service.BillCodeRuleMgrService.saveBillCodeRule(BillCodeRuleVO)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>True</td>
	<td>BillCodeRuleVO</td>
    <td>无 </td>
    <td>要保存的编码规则</td>
  </tr>
</table>

**返回参数说明**  

无


### 获取本规则所有流水依据的最大流水号的API ###

**描述**  

获取本规则所有流水依据的最大流水号，以管理最大流水号  

**请求方法**  

com.yonyou.uap.billcode.service.BillCodeSNmgrService.getMaxSNsByRulePk(String)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>pk_billcodebase</td>
    <td>True</td>
	<td>String</td>
    <td>  </td>
    <td>编码规则主键</td>
  </tr>
</table>

**返回参数说明**  

`List<PubBcrSn>`



### 获取本规则所有返还流水号的API ###

**描述**  

实现业务日志的删除

**请求方法**  

com.yonyou.uap.billcode.service.BillCodeSNmgrService.getRtnCodesByRulePk(String)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>pk_billcodebase</td>
    <td>True</td>
	<td>String</td>
    <td>  </td>
    <td>编码规则主键</td>
  </tr>
</table>


**返回参数说明**  

`List<PubBcrReturn>`  


### 删除返还号API ###

**描述**  

根据返还号ID删除返还号  

**请求方法**  

com.yonyou.uap.billcode.service.BillCodeSNmgrService.delRtnCodeByID(String)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>id</td>
    <td>True</td>
	<td>long</td>
    <td>20 </td>
    <td>返还号ID</td>
  </tr>
</table>  

**返回参数说明**  

无  

### 删除最大流水号API ###

**描述**  

根据最大流水号id删除最大流水号  

**请求方法**  

com.yonyou.uap.billcode.service.BillCodeSNmgrService.delMaxSNByID(String)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>id</td>
    <td>True</td>
	<td>long</td>
    <td>20 </td>
    <td>最大流水号ID</td>
  </tr>
</table>  
**返回参数说明**  

无


### 更新最大流水号API ###

**描述**  

更新最大流水号  

**请求方法**  

com.yonyou.uap.billcode.service.BillCodeSNmgrService.updateMaxSNByID(String, String)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>id</td>
    <td>True</td>
	<td>long</td>
    <td>20 </td>
    <td>最大流水号ID</td>
  </tr>
  <tr>
    <td>sn</td>
    <td>True</td>
	<td>String</td>
    <td>  </td>
    <td>最大流水号</td>
  </tr>
</table>  
**返回参数说明**  

无   


### 获取编码规则上下文API ###

**描述**  

获取编码规则上下文，业务单据号相应字段是否允许修改等信息  

**请求方法**  

com.yonyou.uap.billcode.service.IBillCodeProvider.getBillCodeContext(BillCodeRuleVO)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>True</td>
	<td>BillCodeRuleVO</td>
    <td>  </td>
    <td>编码规则</td>
  </tr>
</table>  

**返回参数说明**

`BillCodeContext：上下文信息`  


### 前编码预取单据号API ###

**描述**  

获取前编码  

**请求方法**  

com.yonyou.uap.billcode.service.IBillCodeProvider.getPreBillCode(BillCodeRuleVO, BillCodeElemInfo, Object)  

**请求方式**  

服务调用   

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>True</td>
	<td>BillCodeRuleVO</td>
    <td>  </td>
    <td>编码规则</td>
  </tr>
  <tr>
    <td>externalElemInfo</td>
    <td>False</td>
	<td>BillCodeElemInfo</td>
    <td>  </td>
    <td>未体现在编码规则中的其他编码元素</td>
  </tr>
  <tr>
    <td>randomCodePara</td>
    <td>False</td>
	<td>Object</td>
    <td>  </td>
    <td>自定义对象，不走编码规则，生成随机码时使用</td>
  </tr>
</table>  
**返回参数说明**  

`单据号`  


### 提交预取的前编码单据编码API ###

**描述**  

提交前编码  

**请求方法**  

com.yonyou.uap.billcode.service.IBillCodeProvider.commitPreBillCode(BillCodeRuleVO, BillCodeElemInfo, String)  

**请求方式**  

服务调用   

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>True</td>
	<td>BillCodeRuleVO</td>
    <td>20 </td>
    <td>编码规则</td>
  </tr>
  <tr>
    <td>billvo</td>
    <td>False</td>
	<td>BillCodeElemInfo</td>
    <td>  </td>
    <td>未体现在编码规则中的其他编码元素</td>
  </tr>
  <tr>
    <td>billCode</td>
    <td>True</td>
	<td>String</td>
    <td>  </td>
    <td>要提交的编码</td>
  </tr>
</table>  

**返回参数说明**  

无  

### 回滚预取单据号API ###

**描述**  

回滚前编码  

**请求方法**  

com.yonyou.uap.billcode.service.IBillCodeProvider.rollbackPreBillCode(BillCodeRuleVO, BillCodeElemInfo, String)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>True</td>
	<td>BillCodeRuleVO</td>
    <td>  </td>
    <td>编码规则</td>
  </tr>
  <tr>
    <td>billvo</td>
    <td>False</td>
	<td>BillCodeElemInfo</td>
    <td>  </td>
    <td>未体现在编码规则中的其他编码元素</td>
  </tr>
  <tr>
    <td>billCode</td>
    <td>True</td>
	<td>String</td>
    <td>  </td>
    <td>要提交的编码</td>
  </tr>
</table>  

**返回参数说明**  
无


### 批量返回num个单据号API ###

**描述**  

获取num个后编码  

**请求方法**  

com.yonyou.uap.billcode.service.IBillCodeProvider.getBatchBillCodes(BillCodeRuleVO, BillCodeBillVO, BillCodeElemInfo, Object, int)

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>True</td>
	<td>BillCodeRuleVO</td>
    <td>  </td>
    <td>编码规则</td>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>False</td>
	<td>BillCodeBillVO</td>
    <td>  </td>
    <td>业务对象VO</td>
  </tr>
  <tr>
    <td>billvo</td>
    <td>False</td>
	<td>BillCodeElemInfo</td>
    <td>  </td>
    <td>未体现在编码规则中的其他编码元素</td>
  </tr>
  <tr>
    <td>randomCodePara</td>
    <td>False</td>
	<td>Object</td>
    <td>  </td>
    <td>自定义对象，不走编码规则，生成随机码时使用</td>
  </tr>
  <tr>
    <td>num</td>
    <td>True</td>
	<td>int</td>
    <td>  </td>
    <td>要取的编号数量</td>
  </tr>
</table>  

**返回参数说明**   

`List<String>,一批单据号`  

### 获取一个生成的单据号API ###

**描述**  

获取一个后编码  

**请求方法**  

com.yonyou.uap.billcode.service.IBillCodeProvider.getBillCode(BillCodeRuleVO, BillCodeBillVO, BillCodeElemInfo, Object)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>True</td>
	<td>BillCodeRuleVO</td>
    <td>  </td>
    <td>编码规则</td>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>False</td>
	<td>BillCodeBillVO</td>
    <td>  </td>
    <td>业务对象VO</td>
  </tr>
  <tr>
    <td>billvo</td>
    <td>False</td>
	<td>BillCodeElemInfo</td>
    <td>  </td>
    <td>未体现在编码规则中的其他编码元素</td>
  </tr>
  <tr>
    <td>randomCodePara</td>
    <td>False</td>
	<td>Object</td>
    <td>  </td>
    <td>自定义对象，不走编码规则，生成随机码时使用</td>
  </tr>
</table>  

**返回参数说明**  

`单据号`  

### 删除单据时回退单据号API ###

**描述**  

回退单据号  

**请求方法**  

com.yonyou.uap.billcode.service.IBillCodeProvider.returnBillCode(BillCodeRuleVO, BillCodeBillVO, BillCodeElemInfo, String)  

**请求方式**  

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
	<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>True</td>
	<td>BillCodeRuleVO</td>
    <td>  </td>
    <td>编码规则</td>
  </tr>
  <tr>
    <td>rulevo</td>
    <td>False</td>
	<td>BillCodeBillVO</td>
    <td>  </td>
    <td>业务对象VO</td>
  </tr>
  <tr>
    <td>billvo</td>
    <td>False</td>
	<td>BillCodeElemInfo</td>
    <td>  </td>
    <td>未体现在编码规则中的其他编码元素</td>
  </tr>
  <tr>
    <td>billCode</td>
    <td>True</td>
	<td>String</td>
    <td>  </td>
    <td>要回退的单据号</td>
  </tr>
</table>  

**返回参数说明**  

无  

### 删除回退的单据号API ###

**描述**  

将已经回退的单据号丢弃  

**请求方法**  

com.yonyou.uap.billcode.service.IBillCodeProvider.DeleteRetrunedBillCode(BillCodeRuleVO, BillCodeElemInfo, String)  

**请求方式**

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th><br>  参数字段<br>  </th>
    <th><br>  必选<br>  </th>
    <th><br>  类型<br>  </th>
    <th><br>  长度限制<br>  </th>
    <th><br>  说明<br>  </th>
  </tr>
  <tr>
    <td><br>  rulevo<br>  </td>
    <td><br>  True<br>  </td>
    <td><br>  BillCodeRuleVO<br>  </td>
    <td></td>
    <td><br>  编码规则<br>  </td>
  </tr>
  <tr>
    <td><br>  billvo<br>  </td>
    <td><br>  False<br>  </td>
    <td><br>  BillCodeElemInfo<br>  </td>
    <td></td>
    <td><br>  未体现在编码规则中的其他编码元素<br>  </td>
  </tr>
  <tr>
    <td><br>  billCode<br>  </td>
    <td><br>  True<br>  </td>
    <td><br>  String<br>  </td>
    <td></td>
    <td><br>  已回退的单据号<br>  </td>
  </tr>
</table>  

**返回参数说明**  

无  

### 删除已经使用的单据号API ###

**描述**  

单据号已经使用，但是在编码规则还提供，这种情况下删除它  

**请求方法**  

com.yonyou.uap.billcode.service.IBillCodeProvider.AbandenBillCode(BillCodeRuleVO, BillCodeBillVO, BillCodeElemInfo, String)  

**请求方式**

服务调用  

**请求参数说明**  

<table>
  <tr>
    <th><br>  参数字段<br>  </th>
    <th><br>  必选<br>  </th>
    <th><br>  类型<br>  </th>
    <th><br>  长度限制<br>  </th>
    <th><br>  说明<br>  </th>
  </tr>
  <tr>
    <td><br>  rulevo<br>  </td>
    <td><br>  True<br>  </td>
    <td><br>  BillCodeRuleVO<br>  </td>
    <td></td>
    <td><br>  编码规则<br>  </td>
  </tr>
  <tr>
    <td><br>  rulevo<br>  </td>
    <td><br>  False<br>  </td>
    <td><br>  BillCodeBillVO<br>  </td>
    <td></td>
    <td><br>  业务对象VO<br>  </td>
  </tr>
  <tr>
    <td><br>  billvo<br>  </td>
    <td><br>  False<br>  </td>
    <td><br>  BillCodeElemInfo<br>  </td>
    <td></td>
    <td><br>  未体现在编码规则中的其他编码元素<br>  </td>
  </tr>
  <tr>
    <td><br>  billCode<br>  </td>
    <td><br>  True<br>  </td>
    <td><br>  String<br>  </td>
    <td></td>
    <td><br>  要丢弃的单据号<br>  </td>
  </tr>
</table>  

**返回参数说明**  
无   
