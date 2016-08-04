# 编码规则 #

## 功能简介 ##

在业务系统中，业务对象通常会有一个**业务对象编号**，业务对象编号作为业务对象具有业务意义的标识，一般由时间，序列号，常量等子段按照一定的规则拼接而成，这个规则我们称为'**编码规则**'，各个子段我们称之为'编码元素'。考虑如下的业务对象编号：'TT-20150615-0001',编码规则为:（常量'TT-'+yyyyMMdd类型的时间'20150615'+常量'-'+序列号'0001'),如果时间变为'20150616',序列号重新从'0001'开始流水，那么我们称此时间编码元素是'流水依据元素',所有的流水依据元素合起来叫做本编码规则的'流水依据'，同一个编码规则，可以有很多个流水依据！  

目前比较少编码规则的开源组件，网上能找到的也只是一些简单的实现类或者设计思路，要么将规则写死在代码中，缺乏通用性和灵活性，要么缺少全面的考虑，本组件提供了一种通用的编码规则解决方案。
本组件的核心是根据编码规则和一些必要信息，生成业务对象编码。组件需要的编码规则可以存储在文件中，甚至是手工构造，或者存储在数据库中。本组件提供了存储在数据库中一种实现，提供了基本的增加，删除，修改编码规则的服务方法，用户可以在此调用此服务，开发出自己的编码规则设计工具，也可以直接设计符合自己要求的编码规则数据库结构和服务，编码规则VO符合组件的模型即可（实现组件的规则接口）。  


## 工作流程 ##

1.	获取编码规则：从自己的编码规则存储位置获取到编码规则。
2.	调用编码规则的服务：根据需要调用相关接口（用户在使用的过程中，可以进行再封装，将获取编码的服务和编码规则接口调用服务封装在一起，方便使用）。  


## API接口 ##

### 获取编码规则API ###

**描述**  
根据编码规则编码查询编码规则  
**请求方法**  

	com.yonyou.uap.billcode.service.BillCodeRuleMgrService.getBillCodeRule(String)  

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

	com.yonyou.uap.billcode.service.BillCodeSNmgrService.delRtnCodeByID(long)  
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

	com.yonyou.uap.billcode.service.BillCodeSNmgrService.delMaxSNByID(long)  
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

	com.yonyou.uap.billcode.service.BillCodeSNmgrService.updateMaxSNByID(long, String)  
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
    <td>未体现在编码规则中的其他编码原始</td>
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
    <td>未体现在编码规则中的其他编码原始</td>
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
    <td>未体现在编码规则中的其他编码原始</td>
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
    <td>未体现在编码规则中的其他编码原始</td>
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
    <td>未体现在编码规则中的其他编码原始</td>
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
    <td>未体现在编码规则中的其他编码原始</td>
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
    <td><br>  未体现在编码规则中的其他编码原始<br>  </td>
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
    <td><br>  未体现在编码规则中的其他编码原始<br>  </td>
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


## 开发步骤 ##

### 引入组件jar包 ###

如果项目是maven工程，可以直接使用如下依赖：

		<!-- 编码规则组件 -->
		<dependency>
      <groupId>com.yonyou.iuap</groupId>
      <artifactId>iuap-billcode</artifactId>
      <version>1.0.0-RELEASE</version>
    </dependency> 


### 执行数据库脚本 ###

依次执行examples项目下sql目录中的dll.sql、index.sql、dml.sql建立数据库并初始化数据。


### spring集成 ###

在Spring的配置文件中添加如下配置（或者是将以下内容放在一个单独的配置文件中，在工程启动时加载此文件）：

	<?xml version="1.0" encoding="utf-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
		xmlns:context="http://www.springframework.org/schema/context"
		xmlns:mvc="http://www.springframework.org/schema/mvc" 
		xmlns:jpa="http://www.springframework.org/schema/data/jpa"
		xmlns:tx="http://www.springframework.org/schema/tx"
		xsi:schemaLocation="http://www.springframework.org/schema/beans  classpath:/org/springframework/beans/factory/xml/spring-beans-3.0.xsd  
	      http://www.springframework.org/schema/context  classpath:/org/springframework/context/config/spring-context-3.0.xsd
	      http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa-1.3.xsd
	      http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
	      http://www.springframework.org/schema/mvc classpath:/org/springframework/web/servlet/configspring-mvc-3.0.xsd"
		default-autowire="byName" default-lazy-init="true">
	
		<!-- 属性文件读入 -->
		<bean id="propertyConfigurer"
			class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
			<property name="locations">
				<list>
					<value>classpath*:config/*.properties</value>
				</list>
			</property>
		</bean>
	
		<bean id="entityManagerFactory"
			class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
			<property name="dataSource" ref="dataSource" />
			<property name="jpaVendorAdapter" ref="hibernateJpaVendorAdapter" />
			<property name="packagesToScan" value="com.yonyou.uap.billcode" />
			<property name="jpaProperties">
				<props>
					<!-- 命名规则 My_NAME->MyName -->
					<prop key="hibernate.ejb.naming_strategy">org.hibernate.cfg.ImprovedNamingStrategy</prop>
				</props>
			</property>
		</bean>
			
		<!-- Spring Data Jpa配置 -->
	 	<jpa:repositories base-package=" com.yonyou.uap.billcode"  transaction-manager-ref="transactionManager" entity-manager-factory-ref="entityManagerFactory"/>
	   
		<!-- Jpa 事务配置 -->
		<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
			<property name="entityManagerFactory" ref="entityManagerFactory"/>
		</bean>
	
		<!-- 使用annotation定义事务 -->
		<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" />
	
		<bean id="hibernateJpaVendorAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
			<property name="databasePlatform">
				<bean factory-method="getDialect" class="org.springside.modules.persistence.Hibernates">
					<constructor-arg ref="dataSource"/>
				</bean>
			</property>
		</bean>
		
		<!-- 数据源定义,使用DBCP数据源 -->
		<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
			destroy-method="close">
			<property name="driverClassName" value="${jdbc.driverClassName}"></property>
			<property name="url" value="${jdbc.url}"></property>
			<property name="username" value="${jdbc.username}"></property>
			<property name="password" value="${jdbc.password}"></property>
			<property name="maxActive" value="${jdbc.pool.maxActive}"></property>
			<property name="maxIdle" value="${jdbc.pool.maxIdle}"></property>
			<property name="maxWait" value="${jdbc.pool.maxWait}"></property>
			<property name="defaultAutoCommit" value="${jdbc.defaultAutoCommit}"></property>
		</bean>

		<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate"
			abstract="false" lazy-init="false" autowire="default">
			<property name="dataSource">
				<ref bean="dataSource" />
			</property>
		</bean>
	
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

