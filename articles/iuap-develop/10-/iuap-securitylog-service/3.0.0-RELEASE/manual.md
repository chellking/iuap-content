# 安全日志服务组件概述 #

## 技术背景 ##

iuap、NC Cloud产品的开发非常迅速，为了保障安全性，需要有对应的日志进行记录，保证有迹可循，这种情况下就需要开发一套可满足国家及行业安全和规范要求的安全日志系统。

## 应用场景 ##

为iuap、NC Cloud产品开发提供安全日记记录的方法及工具。比喻，用户登录，用户认证，金额调整等等，都需要记录用户的轨迹。

## 功能说明 ##

1.	提供独立的安全日志管理；
2.	支持独立的安全数据源；
4.	支持同步记录日志和异步记录日志方式；
5.	日志记录符合W7原则；

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：
```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-securitylog-service</artifactId>
	  <version>${iuap.modules.version}</version>
	  <type>war</type>
	</dependency>
```
${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 技术方案 ##
根据需求，提供了两种方式来满足安全日志系统的使用方式：同步调用、异步调用。根据客户的需求，选择相应的实现方式。

### 同步调用 ###

同步调用时业务系统需要集成iuap-securitylog-local-sdk，日志服务可和业务应用一起部署，也可以分开独立部署，会以war包的形式提供出来。

### 异步调用 ###

异步调用时业务系统需要集成iuap-securitylog-rest-sdk，日志服务与业务应用分开部署，为了保证日志不会丢失，能够准确记录，我们采用分布式的MQ来做中转，用户每一次记录日志都会把日志发送到MQ服务器，在日志服务端，会有一个定时任务去获取日志，然后存储到数据库中。


# 使用说明 #

## 服务端部署说明 ##

服务端部署，部署一个war包

## 组件配置 ##
** 1. 数据库信息配置（securitylog-application.properties）：**

下面几项必须配置
```
    jdbc.url=jdbc:mysql://172.20.14.207:3306/securitylog?useUnicode=true&characterEncoding=gb2312
    jdbc.driverclass=com.mysql.jdbc.Driver
    jdbc.user=root
    jdbc.password=root
```
默认数据库连接池配置,不需要设
```   
    initPoolSize=10
    maxPoolSize=5
```

** 注意，war包中不提供数据库驱动，需要手工将驱动放到war的WEB-INF/lib目录下 **


** 2. 配置文件参考：参考war包中的springDispatcherServlet-servlet.xml和securitylog-applicationContext.xml **

参考war包中的springDispatcherServlet-servlet.xml和securitylog-applicationContext.xml


** 3. 对于异步调用的方式，需要的配置文件。**

配置文件参考war包中的springDispatcherServlet-servlet.xml和securitylog-applicationContext.xml和securitylog-applicationContext-mq-consumer.xml

MQ服务器配置securitylogMQConfig.properties，需要放在classpath目录下：
```
    #集群地址配置，多个的用逗号隔开，
    mq.address=172.20.14.133:5672

    #如果mq.isLocal=true, 可以不用配置下面两项的值
    mq.username=admin
    mq.password=admin
```

** 4. 执行数据库脚本 **

依次执行examples项目下sql目录中的dll.sql、index.sql、dml.sql建立数据库并初始化数据。
