# 电商链接器组件概述 #

## 业务需求 ##

电子商务在现代化社会扮演的角色越来越重要，因此许多应用都需要和电商平台进行对接，那么如何实现这种对接，困扰着不少企业。电商连接器组件能轻松解决这个问题。

## 解决方案 ##

通过调用第三方电商平台的API，来获取电商的数据。
对各个不同的电商平台，提供了一个统一的入口，以统一的JSON格式返回电商平台上可查的数据。

## 功能说明 ##

1. 定义标准接口;
2. 转换Json文件格式;
3. 发送数据标准接口；
4. 接收数据标准接口；

# 整体设计 #

## 依赖环境 ##
```
	<dependency>
    	<groupId>com.yonyou.iuap</groupId>
    	<artifactId>iuap-ecadapter</artifactId>
    	<version>${iuap.modules.version}</version>
	</dependency>
```
${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 工作流程 ##

1. 构造请求数据  
按照各大支持开放API的电商平台的数据格式要求，将所需数据封装成json格式的数据。
2. 发送数据到电商通服务器  
通过HTTP POST发送第一步的数据到电商通平台。  
3. 获取返回数据  
将第二步返回的数据获取。   

## API接口 ##

由于各大平台的API接口参数差异性极大，所以当用户需要调用某一平台的API时，请直接参考对应平台的API说明：   
[淘宝开放平台](https://open.taobao.com/doc2/api_list.htm?spm=a219a.7386653.1.24.HFVMWJ)、[京东开放平台](http://jos.jd.com/api/index.htm)、[苏宁开放平台](http://open.suning.com/ospos/apipage/toApiListMenu.do)......

## 开发步骤 ##


1 . 将配置文件放到指定目录  

**ecadapter-SellerAccountInfo.properties**用来配置卖家在各电商的信息，需要自行修改，此配置文件需要存放在工程的classpath路径下。 (可在示例工程的src/main/resources目录下找到) 

2 . 将组件的配置文件加载到web.xml中 
 
组件的配置文件ecadapter-applicationContext.xml，可在示例工程rc/main/resources目录下找到
在web.xml文件中添加如下配置：   
```
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
    	classpath*:/ecadapter-applicationContext.xml,
    </param-value>
	</context-param>
```
或者将组件提供的servlet配置到工程中：
```
    <servlet>
    	<servlet-name>ecadapter</servlet-name>
    	<servlet-class>com.yonyou.uap.ieop.ecadapter.service.ECAdapterServlet</servlet-class>
    </servlet>
    <servlet-mapping>
    	<servlet-name>ecadapter</servlet-name>
    	<url-pattern>/ecadapter</url-pattern>
    </servlet-mapping>
```

3 . 发送HTTP请求到Servlet或者controller  
在前端可以通过aJax异步请求(或者其他发送HTTP请求的方式)到配置的servlet/controller中，参数格式参见具体电商平台的API接口。
组件将会执行电商连接操作，在返回的data中将是标准的json数据，可以进一步处理。

