#认证组件概述#

## 业务需求 ##
业务系统以Web应用的方式向外提供服务的同时，都需要对用户的请求信息进行验证，确保用户登录和操作的合法性，同时又要避免每次操作都要求用户输入密码，影响用户的体验，所以需要对登录用户的会话信息进行有效的管理和维护。传统的通过HttpSession进行用户信息的存储有诸多弊端，例如在系统集群的时候，需要设置Session粘滞或者Session复制，在服务器宕机时，保存在服务器上面的会话信息都会丢失，相关联用户都需要重新登录。

这就要求提供一种解决方案，提供基础的用户认证登录限制，并统一的管理Session信息，使得Session信息的存储与Web应用服务器无关。

##解决方案##

iuap认证组件通过配置的方式对指定路径的服务资源进行拦截和认证，具体认证可以根据不同的Token生成方式进行。利用分布式缓存组件Redis来存储管理Session信息，从而实现了会话无状态的统一集中，并且与Shiro框架实现集成，将Shiro和Web应用组装，配合分布式缓存，共同实现了Web服务的无状态，更便于服务的水平扩展。

##功能说明##
1.	使用分布式缓存统一管理用户会话；
2.	支持Cookie和Http header的方式验证用户信息；
3.	支持自定义Token生成参数；
4.	支持会话超时时间设置；
5.	支持PC端和Mobile的不同会话管理策略；
6.	支持与门户与第三方登录进行集成。


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖Apache的shiro框架,引入了shiro.shiro-spring的1.2.3版本和iuap平台的一些基础组件如iuap-log和iuap-cache，其对外提供的依赖方式如下：

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-auth</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能结构 ##

<img src="/images/shiro_auth.jpg"/>

**基本概念**

身份验证，即在应用中谁能证明他就是他本人。一般提供如他们的身份ID一些标识信息来表明他就是他本人，如提供身份证，用户名/密码来证明。在shiro中，用户需要提供principals （身份）和credentials（证明）给shiro，从而应用能验证用户身份：

- principals：身份，即主体的标识属性，可以是任何东西，如用户名、邮箱等，唯一即可。一个主体可以有多个principals，但只有一个Primary principals，一般是用户名/密码/手机号。
- credentials：证明/凭证，即只有主体知道的安全值，如密码/数字证书等。
最常见的principals和credentials组合就是用户名/密码了。接下来先进行一个基本的身份认证。
 
另外两个相关的概念是之前提到的Subject及Realm，分别是主体及验证主体的数据源。

## 流程说明 ##

- 1、首先调用Subject.login(token)进行登录，其会自动委托给Security Manager，调用之前必须通过SecurityUtils. setSecurityManager()设置；
- 2、SecurityManager负责真正的身份验证逻辑；它会委托给Authenticator进行身份验证；
- 3、Authenticator才是真正的身份验证者，Shiro API中核心的身份认证入口点，此处可以自定义插入自己的实现；
- 4、Authenticator可能会委托给相应的AuthenticationStrategy进行多Realm身份验证，默认ModularRealmAuthenticator会调用AuthenticationStrategy进行多Realm身份验证；
- 5、Authenticator会把相应的token传入Realm，从Realm获取身份验证信息，如果没有返回/抛出异常表示身份验证失败了。此处可以配置多个Realm，将按照相应的顺序及策略进行访问。

# 使用说明 #

## 组件包说明 ##
iuap-auth组件利用shiro的上述登录认证的步骤，构造了token并进行了认证动作，并提供SessionManager管理类，将传统方式的一些Session信息存储在了分布式缓存Redis中，方便后续的访问。

iuap-auth组件依赖分布式缓存Redis，所以部署时候，需要先安装部署Redis，使用步骤可以参考iuap-cache组件的说明文档。

本组件只提供基础的认证登录过程，shiro中涉及到的权限相关集成，请参考应用组件库中的权限组件相关手册。

##组件配置##

**1:在属性文件中，配置session所需要的redis的连接url、sessionTimeout等信息**

	配置示例：
	
	redis.session.url=direct://localhost:6379?poolSize=50&poolName=mypool
	sessionTimeout=6000
	//是否登录时候剔除当前用户在其他位置的登录,默认为不互踢
	sessionMutex=false

**2:Spring的配置文件中，加入shiro的配置文件，如applicationContext-shiro.xml**

	1）配置statelessRealm

	2）配置TokenProcessor，pc端或者手机端登录的token生成用

	3）shiroFilter和需要过滤的资源url

	4）配置会话管理的sessionJedisPool和sessionMgr


**3:工程中引入对iuap-auth组件的依赖**

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-auth</artifactId>
		<version>${iuap.modules.version}</version>
	</dependency>


${iuap.modules.version} 为平台在maven私服上发布的组件的version,需要在项目的pom.xml中定义。	

**4:代码中调用组件提供的API，生成token等信息并保存cookies**

	TokenParameter tp = new TokenParameter();
	tp.setUserid(userName);
	//设置登录时间
	tp.setLogints(String.valueOf(System.currentTimeMillis()));
	//租户信息
	tp.getExt().put(AuthConstants.PARAM_TENANTID , "tenant01");
	Cookie[] cookies = webTokenProcessor.getCookieFromTokenParameter(tp);
	for(Cookie cookie : cookies){
	    response.addCookie(cookie);
	}

**5:调用组件提供的对session信息的API存取session信息**
	
参考SessionManager中的API。

## 工程样例 ##


<img src="/images/auth_example.jpg"/>

开发工具包DevTool中携带了对认证组件的示例工程，位置位于DevTool/examples/example\_iuap\_auth下，在iuap Studio中导入已有的Maven工程，可以将示例工程导入到工作区。示例工程中有较为完整的对iuap-auth组件的使用示例代码。

## 开发步骤 ##

- 配置示例工程中的redis.session.url为正确的redis地址，redis可以采用DevTool中bin目录下的redis，例如示例工程下的application.properties

		#auth组件需要的redis地址配置
		redis.session.url=direct://localhost:6379?poolSize=50&poolName=mypool
		
		#session过期时间，对应的是redis中的缓存失效时间
		sessionTimeout=6000
		
		#是否登录时候剔除当前用户在其他位置的登录,默认为不互踢
		sessionMutex=false
		
		#客户定义的不进行shiro过滤的url后缀
		filter_excludes=.woff2
		
		#当前应用的context名
		context.name=/example_iuap_auth
		
		#应用的编号
		sysid=hr_cloud

- 配置web.xml

		<filter>
	    	<filter-name>shiroFilter</filter-name>
	    	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	    	<init-param>
	      		<param-name>targetFilterLifecycle</param-name>
	      		<param-value>true</param-value>
	    	</init-param>
	    </filter>
	    <filter-mapping>
	    	<filter-name>shiroFilter</filter-name>
	    	<url-pattern>/*</url-pattern>
	    </filter-mapping>

- 配置applicationContex-shiro.xml,例如示例工程中的配置文件

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns="http://www.springframework.org/schema/beans"
			xmlns:util="http://www.springframework.org/schema/util" xmlns:aop="http://www.springframework.org/schema/aop"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xsi:schemaLocation="
		       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
		       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
		
			<bean id="statelessRealm" class="com.yonyou.iuap.auth.shiro.StatelessRealm">
				<property name="cachingEnabled" value="false" />
			</bean>
		
			<!-- Subject工厂 -->
			<bean id="subjectFactory"
				class="com.yonyou.iuap.auth.shiro.StatelessDefaultSubjectFactory" />
			<bean id="webTokenProcessor" class="com.yonyou.iuap.auth.token.DefaultTokenPorcessor">
				<property name="id" value="web"></property>
				<property name="path" value="${context.name}"></property> 
				<property name="expr" value="${sessionTimeout}"></property>
				<property name="exacts">
					<list>
						<value type="java.lang.String">tenantid</value>
					</list>
				</property>
			</bean>
			<bean id="maTokenProcessor" class="com.yonyou.iuap.auth.token.DefaultTokenPorcessor">
				<property name="id" value="ma"></property>
				<property name="path" value="${context.name}"></property> 
				<property name="expr" value="-1"></property>
				<property name="exacts">
					<list>
						<value type="java.lang.String">tenantid</value>
					</list>
				</property>
			</bean>
		
			<bean id="tokenFactory" class="com.yonyou.iuap.auth.token.TokenFactory">
				<property name="processors">
					<list>
						<ref bean="webTokenProcessor" />
						<ref bean="maTokenProcessor" />
					</list>
				</property>
			</bean>
		 
		
			<!-- 会话管理器 -->
			<bean id="sessionManager" class="org.apache.shiro.session.mgt.DefaultSessionManager">
				<property name="sessionValidationSchedulerEnabled" value="false" />
			</bean>
		
			<!-- 安全管理器 -->
			<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
				<property name="realms">
					<list>
						<ref bean="statelessRealm" />
					</list>
				</property>
				<property name="subjectDAO.sessionStorageEvaluator.sessionStorageEnabled"
					value="false" />
				<property name="subjectFactory" ref="subjectFactory" />
				<property name="sessionManager" ref="sessionManager" />
			</bean>
		
			<!-- 相当于调用SecurityUtils.setSecurityManager(securityManager) -->
			<bean
				class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
				<property name="staticMethod"
					value="org.apache.shiro.SecurityUtils.setSecurityManager" />
				<property name="arguments" ref="securityManager" />
			</bean>
		
			<bean id="statelessAuthcFilter" class="com.yonyou.iuap.auth.shiro.StatelessAuthcFilter">
				<property name="sysid" value="${sysid}" />
				<property name="tokenFactory" ref="tokenFactory" />
			</bean>
		
			<bean id="logout" class="com.yonyou.iuap.auth.shiro.LogoutFilter"></bean>
		
			<!-- Shiro的Web过滤器 -->
			<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
				<property name="securityManager" ref="securityManager" />
				<property name="loginUrl" value="/login" />
				<property name="successUrl" value="/" />
				<property name="filters">
					<util:map>
						<entry key="statelessAuthc" value-ref="statelessAuthcFilter" />
					</util:map>
				</property>
				<property name="filterChainDefinitions">
					<value>
						/logout = logout
						/static/** = anon
						/css/** = anon
						/images/** = anon
						/trd/** = anon
						/js/** = anon
						/api/** = anon
						/cxf/** = anon
						/jaxrs/** = anon
						/** = statelessAuthc
					</value>
				</property>
			</bean>
		
			<!-- Shiro生命周期处理器 -->
			<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
		
			<bean id="sessionJedisPool" class="com.yonyou.iuap.cache.redis.RedisPoolFactory" scope="prototype" factory-method="createJedisPool">
				<constructor-arg value="${redis.session.url}" />
			</bean>
		    
		    <bean id="sessionMgr" class="com.yonyou.iuap.auth.session.SessionManager">
		        <property name="sessionJedisPool" ref="sessionJedisPool"/>
		        <property name="sessionMutex" value="${sessionMutex}"/>
		    </bean>	
		</beans>

关键配置项为shiroFilter，这里配置了那些路径的访问会需要statelessAuthc类型的过滤器拦截，如果特殊的路径不需要拦截，配置成anon。sessionJedisPool和sessionMgr即对redis的配置。

	loginUrl 验证不成功，跳转到此url
	successUrl 登录成功后，跳转到此url

其中webTokenProcessor和maTokenProcessor分别配置的是对浏览器登录和手机端登录的token生成处理bean，其中path和expr即为登录成功后写cookies的一些参数，exacts为业务中需要携带的额外参数，会一并写入到cookies中。示例中例如tenantid参与了token的生成和cookies的回写。在用户登录成功后，这些cookies信息会随着下一次的服务端调用，携带到后端进行验证。

如果是移动端调用，需要登录成功后，将cookies信息以自身的方式进行存储，下次请求发送的时候，以http header的方式携带回来。示例如下：

	Authority:u_logints=1459835806499;u_usercode=hr0001;token=bWEsLTEsSWZFSjBPUkJKMldlM0x5TG82a1R6ckVIVk9WNVVxcm90OTNmRDBRSVhPRzZaT3RpamVsNUxSSjkwR2Q5YVY2enlsc1hRZERpTk5OWkJRcGZLZFNXcmc9PQ;tenantid=yklonf6f

另外，如果是Web端的浏览器的方式访问，可以配置退出的过滤器，配置文件中，退出系统的默认filter如下：

	<bean id="logout" class="com.yonyou.iuap.auth.shiro.LogoutFilter"></bean>

业务信息可以重写此filter，清除自身的其它信息。

- 编写登录处理类，示例中默认的登录示例代码如下：

		package com.yonyou.iuap.auth.example.web;
		
		import java.io.IOException;
		
		import javax.servlet.http.Cookie;
		import javax.servlet.http.HttpServletRequest;
		import javax.servlet.http.HttpServletResponse;
		
		import org.apache.shiro.web.filter.authc.FormAuthenticationFilter;
		import org.slf4j.Logger;
		import org.slf4j.LoggerFactory;
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.stereotype.Controller;
		import org.springframework.ui.Model;
		import org.springframework.web.bind.annotation.RequestMapping;
		import org.springframework.web.bind.annotation.RequestMethod;
		import org.springframework.web.bind.annotation.RequestParam;
		
		import com.yonyou.iuap.auth.session.SessionManager;
		import com.yonyou.iuap.auth.shiro.AuthConstants;
		import com.yonyou.iuap.auth.token.ITokenProcessor;
		import com.yonyou.iuap.auth.token.TokenParameter;
	
		/**
		 * 默认登录逻辑
		 */
		@Controller
		@RequestMapping(value = "/login")
		public class LoginController {
			
		    private final Logger logger = LoggerFactory.getLogger(getClass());
			
			public static final int HASH_INTERATIONS = 1024;
			
		    @Autowired
		    private SessionManager sessionManager;
		
			//为网页版本的登录Controller指定webTokenProcessor 相应的移动的指定为maTokenProcessor
			@Autowired()
			protected ITokenProcessor webTokenProcessor;
			
			@RequestMapping(method = RequestMethod.GET)
			public String login(Model model) {
				return "login";
			}
			
			@RequestMapping(method = RequestMethod.POST,value="formLogin")
			public String formLogin(HttpServletRequest request, HttpServletResponse response, Model model) throws IOException {
		        String userName = request.getParameter("username");
		        String passWord = request.getParameter("password");
				
				if (passWord != null && userName != null) {
					// 模拟用户
					if("admin".equals(userName) && passWord.equals("admin")){
		                
		                TokenParameter tp = new TokenParameter();
		                tp.setUserid(userName);
		                //设置登录时间
		                tp.setLogints(String.valueOf(System.currentTimeMillis()));
		                //租户信息
		                tp.getExt().put(AuthConstants.PARAM_TENANTID , "hr_cloud");
		                Cookie[] cookies = webTokenProcessor.getCookieFromTokenParameter(tp);
		                for(Cookie cookie : cookies){
		            	    response.addCookie(cookie);
		                }
		            } else {
		            	logger.error("用户名密码错误!");
		                model.addAttribute("accounterror", "用户名密码错误!");
		                return "login";
		            }
		            return "redirect";
				} else {
		            model.addAttribute("accounterror", "你输入的用户不存在!");
		            return "login";
				}
			}
			
			@RequestMapping(method = RequestMethod.POST)
			public String fail(@RequestParam(FormAuthenticationFilter.DEFAULT_USERNAME_PARAM) String userName, Model model) {
				model.addAttribute(FormAuthenticationFilter.DEFAULT_USERNAME_PARAM, userName);
				return "login";
			}
		
		}


## 常用接口 ##

- ITokenProcessor

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>方法名</th>
			<th>参数</th>
			<th>返回值</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>getCookieFromTokenParameter(tp)</td>
			<td>TokenParameter token参数类，包含用户id、登录时间ts等</td>
			<td>Cookies[] token相关的Cookies数组</td>
			<td>根据当前的Token参数获取需要存储的cookies，以记录登录成功的token、登录时间等信息</td>
		</tr>
	</tbody>
</table>


- SessionManager

<table style="border-collapse:collapse">
	<tr>
		<th>方法名</th>
		<th>参数</th>
		<th>返回值</th>
		<th>功能说明</th>
	</tr>

	<tr>
		<td>getAllSessionAttrCache</td>
		<td>String sid（模拟Http的Session的id，Session信息的唯一标识，可以用token代替）</td>
		<td>Map<String, Object>（session对应的map集合）</td>
		<td>从redis中获取指定session下的所有值</td>
	</tr>
	<tr>
		<td>removeSessionCache</td>
		<td>String sid（模拟Http的Session的id，Session信息的唯一标识，可以用token代替）</td>
		<td>void</td>
		<td>从redis中清除session信息</td>
	</tr>
	<tr>
		<td>putSessionCacheAttribute</td>
		<td>String sid（session id）, String key（属性名）, T value（属性值）</td>
		<td><T extends Serializable> 对应value类型的返回对象</td>
		<td>向redis中放置session中的某一项属性信息</td>
	</tr>
	<tr>
		<td>putSessionCacheAttribute</td>
		<td>String sid（session id）, String key（属性名）, T value（属性值）,int timeout(此session信息的reids缓存过期时间)</td>
		<td><T extends Serializable> 对应value类型的返回对象</td>
		<td>从redis中获取session中的某一项属性信息同时设置session的有效期</td>
	</tr>
	<tr>
		<td>updateSessionCacheAttribute</td>
		<td>String sid（session id）, String key（属性名）, T value（属性值）,int timeout(此session信息的reids缓存过期时间)</td>
		<td><T extends Serializable> 对应value类型的返回对象</td>
		<td>更新session中的某一项属性值，并设置session过期时间</td>
	</tr>
	<tr>
		<td>getSessionCacheAttribute</td>
		<td>String sid（session id）, String key（属性名）</td>
		<td>Object 对应value类型的返回对象</td>
		<td>获取session中对应属性名为key的属性值</td>
	</tr>
	<tr>
		<td>removeSessionCacheAttribute</td>
		<td>String sid（session id）, String key（属性名）</td>
		<td>void</td>
		<td>清除session中对应属性名为key的信息</td>
	</tr>
	<tr>
		<td>registOnlineSession</td>
		<td>final String userid（用户标识）, final String token（token登录时候生成的值）, final ITokenProcessor processor（token处理类，shiro的配置文件中注入）</td>
		<td>void</td>
		<td>注册登录信息，登录处理类获取cookies时候会调用</td>
	</tr>
	<tr>
		<td>validateOnlineSession</td>
		<td>final String userid（用户标识）, final String token（token登录时候生成的值）</td>
		<td>boolean 是否有效标识</td>
		<td>判断传入的token是否有效</td>
	</tr>
	<tr>
		<td>delOnlineSession</td>
		<td>final String userid（用户标识）, final String token（token登录时候生成的值）</td>
		<td>void</td>
		<td>退出登录时候，删除在线的session信息</td>
	</tr>
	<tr>
		<td>deleteUserSession</td>
		<td>final String userid（用户标识）</td>
		<td>void</td>
		<td>禁用或者删除用户时，删除用户相关的信息</td>
	</tr>
</table>

