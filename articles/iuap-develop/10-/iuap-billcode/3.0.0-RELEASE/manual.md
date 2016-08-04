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

在Spring的配置文件中添加如下配置（或者是将以下内容放在一个单独的配置文件中，在工程启动时加载此文件）：

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

### 使用API接口完成请求 ###

按照API接口 要求，调用对应API，管理编码规则或者生成编码

### 扩展机制 ###

编码规则组件支持对编码规则元素的自定义替换，通过上下文配置文件来实现自定义的编码规则组件设置（位于jar包'com/yonyou/uap/billcode/BillCodeEngineContext.xml'），如下

![](image/image001.jpg)    

支持以下几种扩展方式：  

**一、自定义编码元素处理器**  
1. 复制本jar包中  `'com/yonyou/uap/billcode/BillCodeEngineContext.xml'到任意'类路径''A'`
2. 实现编码元素处理器接口：  `'com.yonyou.uap.billcode.elemproc.itf.IElemProcessor'`  
3. 在编码规则引擎上下文配置文件（复制的`'BillCodeEngineContext.xml'`）中`'添加'`或`'替换'`元素处理器

**二、自定义序列号编码元素的序列号生成方式**  
1.同上  
2.实现编码元素处理器接口：      `'com.yonyou.uap.billcode.sngenerator.ISNGenerator'`  
3.在编码规则引擎上下文配置文件（复制的`'BillCodeEngineContext.xml'`）中`'添加'`或`'替换'`序列号生成器

**三、自定义系统时间编码元素的时间获取方式**  
1.同上  
2.实现编码元素处理器接口：      `'com.yonyou.uap.billcode.sysdate.ISysDateProvider'`  
3.在编码规则引擎上下文配置文件（复制的`'BillCodeEngineContext.xml'`）中`'替换'`时间获取方式

**四、自定义唯一性随机编码生成类**（不需要业务对象编号有业务意义时，此类提供唯一性的无业务意义的编码）  
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
	<td>true/false</td>
  </tr>
  <tr>
    <td>isBolSNAppendZero</td>
    <td>序列号是否补零</td>
	<td>true/false，不补0，序列号4位，1显示为1，否则，为0001</td>
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
	<td>见常量com.yonyou.uap.billcode.model.IBillCodeElemVO.TYPE_XX</td>
  </tr>
  <tr>
    <td>elemvalue</td>
    <td>元素值</td>
	<td>不同类型的编码元素意义不同：
	 <br>0-流水号 			：流水号生成器的类型
	 <br>1-常量     			：常量值
	 <br>2-系统时间		：空
	 <br>3-随机码			：空
	 <br>4-单据业务时间	：扩展人员决定是否使用
	 <br>5-单据字符串属性     ：扩展人员决定是否使用
	 <br>6-单据参照属性	：扩展人员决定是否使用</td>
  </tr>
  <tr>
    <td>elemlenth</td>
    <td>编码元素的长度</td>
	<td></td>
  </tr>
  <tr>
    <td>isrefer</td>
    <td>元素流水依据</td>
	<td>见常量com.yonyou.uap.billcode.model.IBillCodeElemVO.REF_XXX</td>
  </tr>
  <tr>
    <td>eorder</td>
    <td>编码元素的排序</td>
	<td>最终的单据编码按此顺序拼接而成</td>
  </tr>
  <tr>
    <td>fillstyle</td>
    <td>补位方式</td>
	<td>见常量com.yonyou.uap.billcode.model.IBillCodeElemVO.FIllSTYLE_XXX</td>
  </tr>
  <tr>
    <td>fillsign</td>
    <td>补位符号</td>
	<td></td>
  </tr>
  <tr>
    <td>datedisplayformat</td>
    <td>日前格式化</td>
	<td>只有元素是日期类型时才会用到，最终日期的格式</td>
  </tr>
</table>


## API接口 ##

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
