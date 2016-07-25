# 组件简介 #

## 业务需求 ##
业务开发过程中经常会用到一些共同的常用操作，平台提供公共类可以提高开发效率和统一优化。

## 解决方案 ##
iuap工具包组件主要封装了一些常用的工具类，如属性文件的读取，Http请求调用和上下文传递等，供业务人员开发功能时调用，并对这些工具类进行调优，提高性能。

##功能说明##
1.	支持对属性文件的读取；
2.	支持对Http请求调用的封装、自定义头；
3.	支持Http上下文信息的传递；
4.	支持对配置文件敏感信息的加密;

#使用说明#

##Maven依赖##

**工程的pom.xml中，增加对iuap-utils的依赖,如果其它组件已经依赖uitls，可省略此步骤**


	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-utils</artifactId>
		<version>${iuap.modules.version}</version>
	</dependency>	

iuap.modules.version为在pom.xml中定义的需要引用的组件的版本。

##属性文件读取的使用##

	//配置属性文件application.properties,此文件可以放置在classpath中，也可放置在制定的磁盘目录/etc/iuap下
	public static String getPropertyByKey(String key);

工具类的默认取值顺序为，系统系统属性 > 环境变量 > 属性配置文件

注意：如果想通过参数的方式指定属性文件的位置，可使用-Diuap.server.conf.url进行配置，但是-D方式会影响整个JVM参数，如果只想本工程生效，需要调用工具类的静态方法：

	public static void setConfFileUrl(String confFileUrl)

## http工具类使用 ##

HttpUtil类中封装了对HTTP的get请求、post请求的基本调用，可以添加自定义的header

	/**
	 * 基本的Post请求
	 * @param url 请求url
	 * @param queryParams 请求头的查询参数
	 * @param formParams post表单的参数
	 */
	public HttpResponse doPost(String url, Map<String, String> queryParams, Map<String, String> formParams);

	/**
	 * 加入header的Get请求
	 * @param url 请求url
	 * @param queryParams 请求头的查询参数
	 * @param headers http header
	 */
	public HttpResponse doGet(String url, Map<String, String> queryParams, Map<String, String> headers);


## http请求传递上下文信息 ##

HttpContextUtil 类在 HttpUtil的基础上，可以传递上下文信息到被调用的系统中，上下文信息被自动放到header头里面。

调用方法和HttpUtil类似：

	/**
	 * 加入header的Get请求
	 * @param url 请求url
	 * @param queryParams 请求头的查询参数
	 * @param headers http header
	 * @return HttpResponse
	   
	 */
	public HttpResponse doGetWithContext(String url, Map<String, String> queryParams, Map<String, String> headers) throws Exception{
		headers = processContextHeaders(headers);
		return httpUtil.doGet(url, queryParams, headers);
	}	


	/**
	 * 基本的Post请求
	 * @param url 请求url
	 * @param queryParams 请求头的查询参数
	 * @param formParams post表单的参数
	 * @return HttpResponse
	 */
	public HttpResponse doPostWithContext(String url, Map<String, String> queryParams, Map<String, String> formParams) throws Exception{
		return doPostWithContext(url, queryParams, formParams, null);
	}


发送的上下文信息字段包括


  	String sysid;  //系统id
  	String tenantid ;// 租户id
  	String userid;    //用户id
  	String callid;    //被调用id
 	String token ;    //令牌信息
  	String theme;     //主题
  	String locale;    //本地信息
  	String logints ；//登录时间
 	Map<String, String>  parameters ; //扩展属性，自定义信息加入其中


这些信息放在java 的  ThreadLocal里面 。

接收方配置在 web.xml中配置过滤器来接收传递过来的上下文信息，配置如下：


	<!-- 过滤上下文信息 -->
	<filter>    
		<filter-name>InvocationInfoFilter</filter-name>    
		<filter-class>com.yonyou.iuap.context.filter.ContextFilter</filter-class>  
	</filter>    
	<filter-mapping>    
		<filter-name>InvocationInfoFilter</filter-name>    
		<url-pattern>/restcontext/*</url-pattern>    
	</filter-mapping> 


com.yonyou.iuap.context.filter.ContextFilter 类位于  iuap-generic.jar 包中。

<url-pattern>/restcontext/*</url-pattern> 指需要过滤的 url路径。

接收到信息后，调用 InvocationInfoProxy将信息打印出来

		System.out.println( InvocationInfoProxy.getCallid());
		System.out.println( InvocationInfoProxy.getLocale());
		System.out.println( InvocationInfoProxy.getSysid());
		.......

InvocationInfoProxy (位于iuap-generic.jar中)


## 配置文件敏感属性加密 ##

对  application.properties中如用户名，密码等数据加密。

先用 com.yonyou.iuap.utils.PropertyCodec（iuap-util.jar中）中的  encryptPwd() 方法加密；

加密后内容如下：

	jdbc.driver=org.postgresql.Driver
	jdbc.url=eJzLSklKtirILy5JL0otLsyx0tc3NDfSMzLQMzTRMzO0MjUxNtKvqigGAP96C6g=
	jdbc.username=postgres
	jdbc.password=eJwryC8uSS9KLQYAD7YDeA==

applicationContext.xml文件中读取 配置文件

	 <bean id="propertyConfigurer" class="com.yonyou.iuap.config.PropertyConfigurer">
        <property name="encryptKeySet">
            <set> <!-- 配置需要解密的属性-->
                <value>jdbc.password</value>
                <value>jdbc.url</value>
            </set>
        </property>
        <property name="locations">
            <list>
                <value>classpath:application.properties</value>
            </list>
        </property>
        <property name="systemPropertiesMode"
                  value="#{T(org.springframework.beans.factory.config.PropertyPlaceholderConfigurer).SYSTEM_PROPERTIES_MODE_OVERRIDE}"/>
    </bean>
