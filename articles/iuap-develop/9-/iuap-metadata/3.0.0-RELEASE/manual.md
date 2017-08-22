#元数据服务组件概述#

## 业务需求 ##

业务系统在进行模型驱动开发时，系统一开始我们就首先确立实体模型Entity Model，以及它们之间的关系，进而可以交由程序员分别实现表现层、业务服务层和持久层，通过使用元数据设计器，结合NC的现状，注重实效，尽可能提高NC产品的整合力度，减少开发人员的重复、琐碎劳动，提高开发效率，使开发人员能在自己熟悉的应用领域发挥更多的作用，减少技术上的纠缠，从而正确无误地、且快速高质量地完成一个软件开发过程。


## 解决方案##

iuap-mdpersistence和iuap-mdspi组件提供高性能的基于mybatis的元数据模型信息发布、查询、删除和修改功能，提供基于BMF文件的发布方式，对元数据模型信息持久化到数据库中。支持带缓存的查询模型信息，并提供无状态的方式与iuap-mdjdbc组件继承，实现元数据和业务数据的管理。

## 功能说明 ##

1.	查询访问基于元数据的业务对象模型，包括实体、属性和关系；
2.	支持元数据发布到目标数据源；
3.	支持运行时的动态调整业务对象模型，包括新建子实体、关系，以及更改名称等；
4.	支持适用于多租户应用架构的多数据源配置与访问；
5.	支持元数据模型的缓存。

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖MyBatis框架,引入了shiro.shiro-spring的1.2.3版本和iuap平台的一些基础组件如iuap-log和iuap-mybatis，和SaaS平台的组件如iuap-saas-dynamicds、iuap-saas-cache，以及基础的数据库连接组件如postgresql、mysql-connector-java。其对外提供的依赖方式如下：

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-mdpersistence</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-mdspi</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能结构 ##

<img src="/articles/iuap-develop/9-/iuap-metadata/3.1.0-RELEASE/images/structure.JPG"/>

**基本概念**

1. 	**组件**：组件是部署的基本单元 ，包含一个或多个实体、枚举、业务接口；
2. 	**实体**：是指需要有唯一标识的业务对象，NC原有的大部分VO基本上都算得上是实体 ；
3. 	**属性**：是指业务对象的某个性质的定义内容，类似java对象的属性；
4. 	**枚举**：是指可穷举的用于防止用户提供无效值的变量；
5. 	**业务接口**：是指某一类具有相同特性的属性的抽象，可由多个实体继承。


## 流程说明 ##

- 1、首先调用ServiceFinder.findService(ServiceName)以获得需要的服务接口，其会自动查找服务接口的实现类以及管理缓存信息；
- 2、使用获取到的服务接口进行各项操作，如发布、查询元数据信息、删除。

# 使用说明 #

## 组件包说明 ##
iuap平台提供元数据服务以实现元数据的查询、发布、删除和修改等操作。实现了统一的业务模型信息抽取，提供了一套模型驱动的需求、设计、代码概念一致性的解决方案.

##组件配置##

**1:在属性文件中，配置redis的连接url和数据库连接属性**

	示例：
```
	redis.url=direct://localhost:6379?poolName=mypool&poolSize=100
	jdbc.driver=com.mysql.jdbc.Driver
	jdbc.url=jdbc:mysql://localhost:3306/tenant01?useUnicode=true&characterEncoding=utf-8
	jdbc.catalog=tenant01
	jdbc.username=root
	jdbc.password=1234
```	
	

**2:Spring的配置文件中，调整如下配置**

```		
	<context:component-scan base-package="com.yonyou.metadata.mybatis.service">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>

	<bean id="saasCacheManager" class="com.yonyou.iuap.cache.SaasCacheManager">
		<property name="cacheManager" ref="cacheManager" />
	</bean>
	<bean id="metadataCache" class="com.yonyou.metadata.mybatis.util.MetadataCache">
		<property name="saasCacheMgr" ref="saasCacheManager" />
		<property name="cacheManager" ref="cacheManager" />
	</bean>	
```	

**3:工程中引入对iuap-metadata组件的依赖**

```	
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-mdpersistence</artifactId>
	  <version>3.0.0-RELEASE</version>
	</dependency>
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-mdspi</artifactId>
	  <version>3.0.0-RELEASE</version>
	</dependency>	
```	

**4:代码中调用组件提供的API，操作元数据**

```	
	/**
	 * 测试通过组件ID获取名称空间
	 * 
	 * @throws Exception
	 */
	@Test
	 public void testgetNameSpaceByCompID() throws Exception {

    	InvocationInfoProxy.setTenantid("publishtest");
   		AbstractMetadataService service = (AbstractMetadataService) ServiceFinder.findMetadataService();
   		String en = service.getNameSpaceByCompID("05ab4927-03de-4fd0-b118-6f65df6378d0", null);
   		Assert.assertNotNull(en);

  }
```	


**5:更多API操作和配置方式，请参考缓存对应的示例工程(DevTool/examples/example_iuap_metadata)**

## 工程样例 ##


<img src="/articles/iuap-develop/9-/iuap-metadata/3.1.0-RELEASE/images/metadata_example.jpg"/>

开发工具包DevTool中携带了对元数据服务的示例工程，位置位于DevTool/examples/example\_iuap\_metadata下，在IUAP_STUDIO中导入已有的Maven工程，可以将示例工程导入到工作区。示例工程中有较为完整的对iuap-mdpersistence和mdspi组件的使用示例代码。

## 开发步骤 ##

- 配置示例工程中的redis.session.url为正确的redis地址，redis可以采用DevTool中bin目录下的redis，例如示例工程下的application.properties

		#元数据服务组件需要的redis地址配置
		redis.url=direct://localhost:6379?poolSize=50&poolName=mypool
		
		#数据库配置信息
		jdbc.driver=org.postgresql.Driver
		jdbc.url=jdbc:postgresql://localhost:5432/publishtest?useUnicode=true&characterEncoding=utf-8
		jdbc.catalog=publishtest
		jdbc.username=root
		jdbc.password=
		
		#连接池配置信息
		jdbc.pool.maxIdle=10
		jdbc.pool.maxActive=100
		jdbc.pool.maxWait=120000
		jdbc.pool.initialSize=20	
		jdbc.pool.minEvictableIdleTimeMillis=6000
		jdbc.pool.removeAbandoned=true
		jdbc.pool.removeAbandonedTimeout=6000


- 配置applicationContex-***.xml,例如示例工程中的配置文件

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:aop="http://www.springframework.org/schema/aop"
		xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"
		xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
		http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd"
		default-lazy-init="true">

		<description>saas cache</description>
	
		<!-- 说明，如果想为业务配置多份的cache，需要配置多个pool，连接多个不同的url。 用bean id区分并注入，避免采用@Autowired的方式按类型注入。 -->
	
	
		<bean id="redisPool" class="com.yonyou.iuap.cache.redis.RedisPoolFactory"
			scope="prototype" factory-method="createJedisPool">
			<constructor-arg value="${redis.url}" />
		</bean>
	
		<bean id="jedisTemplate" class="org.springside.modules.nosql.redis.JedisTemplate">
			<constructor-arg ref="redisPool"></constructor-arg>
		</bean>
	
		<bean id="redisShardedPool" class="com.yonyou.iuap.cache.redis.RedisPoolFactory"
			scope="prototype" factory-method="createShardedJedisPools">
			<constructor-arg value="${redis.shardedurl}" />
		</bean>
	
		<bean id="jedisShardedTemplate" class="org.springside.modules.nosql.redis.JedisShardedTemplate">
			<constructor-arg ref="redisShardedPool"></constructor-arg>
		</bean>
	
		<bean id="cacheManager" class="com.yonyou.iuap.cache.CacheManager">
			<property name="jedisTemplate" ref="jedisTemplate" />
		</bean>
	
		<bean id="saasCacheManager" class="com.yonyou.iuap.cache.SaasCacheManager">
			<property name="cacheManager" ref="cacheManager" />
		</bean>
		<bean id="metadataCache" class="com.yonyou.metadata.mybatis.util.MetadataCache">
			<property name="saasCacheMgr" ref="saasCacheManager" />
			<property name="cacheManager" ref="cacheManager" />
		</bean>
		</beans>

## 常用接口 ##

- MetadataService

**5:更多API操作和配置方式，请参考缓存对应的示例工程(DevTool/examples/example_iuap_metadata)**

## 工程样例 ##


<img src="/articles/iuap-develop/9-/iuap-metadata/3.1.0-RELEASE/images/metadata_example.jpg"/>

开发工具包DevTool中携带了对元数据服务的示例工程，位置位于DevTool/examples/example\_iuap\_metadata下，在iuap Studio中导入已有的Maven工程，可以将示例工程导入到工作区。示例工程中有较为完整的对iuap-mdpersistence和mdspi组件的使用示例代码。

## 开发步骤 ##

- 配置示例工程中的redis.session.url为正确的redis地址，redis可以采用DevTool中bin目录下的redis，例如示例工程下的application.properties

		#元数据服务组件需要的redis地址配置
		redis.url=direct://localhost:6379?poolSize=50&poolName=mypool
		
		#数据库配置信息
		jdbc.driver=org.postgresql.Driver
		jdbc.url=jdbc:postgresql://localhost:5432/publishtest?useUnicode=true&characterEncoding=utf-8
		jdbc.catalog=publishtest
		jdbc.username=root
		jdbc.password=
		
		#连接池配置信息
		jdbc.pool.maxIdle=10
		jdbc.pool.maxActive=100
		jdbc.pool.maxWait=120000
		jdbc.pool.initialSize=20	
		jdbc.pool.minEvictableIdleTimeMillis=6000
		jdbc.pool.removeAbandoned=true
		jdbc.pool.removeAbandonedTimeout=6000


- 配置applicationContex-***.xml,例如示例工程中的配置文件

		<?xml version="1.0" encoding="UTF-8"?>
		<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
		xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:aop="http://www.springframework.org/schema/aop"
		xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"
		xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
		http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd"
		default-lazy-init="true">

		<description>saas cache</description>
	
		<!-- 说明，如果想为业务配置多份的cache，需要配置多个pool，连接多个不同的url。 用bean id区分并注入，避免采用@Autowired的方式按类型注入。 -->
	
	
		<bean id="redisPool" class="com.yonyou.iuap.cache.redis.RedisPoolFactory"
			scope="prototype" factory-method="createJedisPool">
			<constructor-arg value="${redis.url}" />
		</bean>
	
		<bean id="jedisTemplate" class="org.springside.modules.nosql.redis.JedisTemplate">
			<constructor-arg ref="redisPool"></constructor-arg>
		</bean>
	
		<bean id="redisShardedPool" class="com.yonyou.iuap.cache.redis.RedisPoolFactory"
			scope="prototype" factory-method="createShardedJedisPools">
			<constructor-arg value="${redis.shardedurl}" />
		</bean>
	
		<bean id="jedisShardedTemplate" class="org.springside.modules.nosql.redis.JedisShardedTemplate">
			<constructor-arg ref="redisShardedPool"></constructor-arg>
		</bean>
	
		<bean id="cacheManager" class="com.yonyou.iuap.cache.CacheManager">
			<property name="jedisTemplate" ref="jedisTemplate" />
		</bean>
	
		<bean id="saasCacheManager" class="com.yonyou.iuap.cache.SaasCacheManager">
			<property name="cacheManager" ref="cacheManager" />
		</bean>
		<bean id="metadataCache" class="com.yonyou.metadata.mybatis.util.MetadataCache">
			<property name="saasCacheMgr" ref="saasCacheManager" />
			<property name="cacheManager" ref="cacheManager" />
		</bean>
		</beans>

## 常用接口 ##

- MetadataService

<table style="border-collapse:collapse">
<tr>
<th class="colFirst" scope="col">限定符和类型</th>
<th class="colLast" scope="col">方法和说明</th>
</tr>
<tr id="i1" class="rowColor">
<td class="colFirst"><code>java.util.Map&lt;java.lang.String,Component&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink">batchGetComponent</span>(java.util.List&lt;java.lang.String&gt;&nbsp;componentIDList,
                 java.lang.String&nbsp;tenantID)</code>&nbsp;</td>
</tr>
<tr id="i2" class="altColor">
<td class="colFirst"><code>java.util.Map&lt;java.lang.String,java.lang.Integer&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink">batchGetCompVersion</a></span>(java.util.List&lt;java.lang.String&gt;&nbsp;componentIDList,
                   java.lang.String&nbsp;tenantID)</code>&nbsp;</td>
</tr>
<tr id="i3" class="rowColor">
<td class="colFirst"><code>java.util.List&lt;Entity</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#batchGetEntities-java.util.List-java.lang.String-">batchGetEntities</a></span>(java.util.List&lt;java.lang.String&gt;&nbsp;entityIDList,
                java.lang.String&nbsp;tenantID)</code>
<div class="block"><span class="deprecatedLabel">已过时。</span>&nbsp;</div>
</td>
</tr>
<tr id="i4" class="altColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Enumerate.html" title="com.yonyou.metadata.spi中的类">Enumerate</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#batchGetEnumerates-java.util.List-java.lang.String-">batchGetEnumerates</a></span>(java.util.List&lt;java.lang.String&gt;&nbsp;enumerateIDList,
                  java.lang.String&nbsp;tenantID)</code>
<div class="block"><span class="deprecatedLabel">已过时。</span>&nbsp;</div>
</td>
</tr>
<tr id="i5" class="rowColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Component.html" title="com.yonyou.metadata.spi中的类">Component</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getAllComponents-java.lang.String-java.lang.String-">getAllComponents</a></span>(java.lang.String&nbsp;modulename,
                java.lang.String&nbsp;tenantID)</code>
<div class="block">根据模块名查询该模块里所有的组件</div>
</td>
</tr>
<tr id="i6" class="altColor">
<td class="colFirst"><code>java.util.List&lt;Entity</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getAllEntity-java.lang.String-">getAllEntity</a></span>(java.lang.String&nbsp;tenantID)</code>
<div class="block"><span class="deprecatedLabel">已过时。</span>&nbsp;</div>
</td>
</tr>
<tr id="i7" class="rowColor">
<td class="colFirst"><code>java.util.Map&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>,Entity</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getAllREFEntity-java.lang.String-java.lang.String-java.lang.String-">getAllREFEntity</a></span>(java.lang.String&nbsp;namespace,
               java.lang.String&nbsp;entityName,
               java.lang.String&nbsp;tenantID)</code>
<div class="block">查询所有的REF该（entityName）的所有的Entity的方法，key是REF该Entity的Attribute,value是Attribute的Entity</div>
</td>
</tr>
<tr id="i8" class="altColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getAttributeByClassID-java.lang.String-java.lang.String-">getAttributeByClassID</a></span>(java.lang.String&nbsp;classid,
                     java.lang.String&nbsp;tenantID)</code>
<div class="block">通过全限定名（完整类名）查询其所有的属性方法，</div>
</td>
</tr>
<tr id="i9" class="rowColor">
<td class="colFirst"><code>java.util.Map&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>,Entity</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getAttributeEntityByFK-java.lang.String-java.lang.String-java.lang.String-java.lang.String-">getAttributeEntityByFK</a></span>(java.lang.String&nbsp;namespace,
                      java.lang.String&nbsp;name,
                      java.lang.String&nbsp;fkName,
                      java.lang.String&nbsp;tenantID)</code>
<div class="block">通过子表的类名和外键名查询其外键Attribute 及主表Entity</div>
</td>
</tr>
<tr id="i10" class="altColor">
<td class="colFirst"><code>java.util.List&lt;java.lang.String&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getBizInterfaceNames-java.lang.String-java.lang.String-">getBizInterfaceNames</a></span>(java.lang.String&nbsp;entityID,
                    java.lang.String&nbsp;tenantID)</code>
<div class="block">查询某实体实现的所有接口</div>
</td>
</tr>
<tr id="i11" class="rowColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/BusiItAttr.html" title="com.yonyou.metadata.spi中的类">BusiItAttr</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getBusiItfAttrs-java.lang.String-java.lang.String-java.lang.String-">getBusiItfAttrs</a></span>(java.lang.String&nbsp;namespace,
               java.lang.String&nbsp;name,
               java.lang.String&nbsp;tenantID)</code>&nbsp;</td>
</tr>
<tr id="i12" class="altColor">
<td class="colFirst"><code>java.util.Map&lt;java.lang.String,java.util.Map&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>,<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>&gt;&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getChildTableNameAndImpAtts-java.lang.String-java.lang.String-java.lang.String-java.lang.String-">getChildTableNameAndImpAtts</a></span>(java.lang.String&nbsp;namespace,
                           java.lang.String&nbsp;name,
                           java.lang.String&nbsp;fkName,
                           java.lang.String&nbsp;tenantID)</code>
<div class="block"><span class="deprecatedLabel">已过时。</span>&nbsp;</div>
</td>
</tr>
<tr id="i13" class="rowColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/Component.html" title="com.yonyou.metadata.spi中的类">Component</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getComponentByID-java.lang.String-java.lang.String-">getComponentByID</a></span>(java.lang.String&nbsp;componentID,
                java.lang.String&nbsp;tenantID)</code>
<div class="block">根据组件id 获取组件</div>
</td>
</tr>
<tr id="i14" class="altColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/Component.html" title="com.yonyou.metadata.spi中的类">Component</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getComponentByName-java.lang.String-java.lang.String-">getComponentByName</a></span>(java.lang.String&nbsp;componentname,
                  java.lang.String&nbsp;tenantID)</code>
<div class="block">根据组件名称获取组件</div>
</td>
</tr>
<tr id="i15" class="rowColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/Component.html" title="com.yonyou.metadata.spi中的类">Component</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getComponentByTypeName-java.lang.String-java.lang.String-">getComponentByTypeName</a></span>(java.lang.String&nbsp;name,
                      java.lang.String&nbsp;tenantID)</code>
<div class="block">通过实体Name得到组件</div>
</td>
</tr>
<tr id="i16" class="altColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Component.html" title="com.yonyou.metadata.spi中的类">Component</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getComponents-java.lang.String-">getComponents</a></span>(java.lang.String&nbsp;tenantID)</code>
<div class="block">获取该tenantID下的所有组件</div>
</td>
</tr>
<tr id="i17" class="rowColor">
<td class="colFirst"><code>java.lang.String</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getDDL-java.lang.String-java.lang.String-">getDDL</a></span>(java.lang.String&nbsp;tableName,
      java.lang.String&nbsp;tenantID)</code>&nbsp;</td>
</tr>
<tr id="i18" class="altColor">
<td class="colFirst"><code>Entity</a>[]</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEntities-java.lang.String:A-java.lang.String-">getEntities</a></span>(java.lang.String[]&nbsp;entityIDs,
           java.lang.String&nbsp;tenantID)</code>&nbsp;</td>
</tr>
<tr id="i19" class="rowColor">
<td class="colFirst"><code>java.util.Map&lt;java.lang.String,java.util.List&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEntitiesAEnumerateByCmpID-java.lang.String-java.lang.String-">getEntitiesAEnumerateByCmpID</a></span>(java.lang.String&nbsp;componentID,
                            java.lang.String&nbsp;tenantID)</code>
<div class="block">通过组件的ID获取其所有的Entities</div>
</td>
</tr>
<tr id="i20" class="altColor">
<td class="colFirst"><code>Entity</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEntity-java.lang.String-java.lang.String-java.lang.String-">getEntity</a></span>(java.lang.String&nbsp;namespace,
         java.lang.String&nbsp;entityName,
         java.lang.String&nbsp;tenantID)</code>
<div class="block">根据组件名和Entity名查询相应的Entity</div>
</td>
</tr>
<tr id="i21" class="rowColor">
<td class="colFirst"><code>java.util.Map&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>,Entity</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEntityAndFK-java.lang.String-java.lang.String-java.lang.String-">getEntityAndFK</a></span>(java.lang.String&nbsp;namespace,
              java.lang.String&nbsp;entityClassName,
              java.lang.String&nbsp;tenantID)</code>
<div class="block">通过主表查子表，key是外键</div>
</td>
</tr>
<tr id="i22" class="altColor">
<td class="colFirst"><code>java.util.Map&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>,Entity</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEntityByFK-java.lang.String-java.lang.String-java.lang.String-">getEntityByFK</a></span>(java.lang.String&nbsp;namespace,
             java.lang.String&nbsp;name,
             java.lang.String&nbsp;tenantID)</code>
<div class="block">通过子表的全限定名查主表(一个BMF文件只能有一个主表,所以不需要子表的外键名) 返回外键和主表Entity</div>
</td>
</tr>
<tr id="i23" class="rowColor">
<td class="colFirst"><code>Entity</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEntityByFK-java.lang.String-java.lang.String-java.lang.String-java.lang.String-">getEntityByFK</a></span>(java.lang.String&nbsp;namespace,
             java.lang.String&nbsp;entityName,
             java.lang.String&nbsp;fkName,
             java.lang.String&nbsp;tenantID)</code>
<div class="block">通过表的全限定名和属性（外键）名获取其对应的实体并且在其property文件中保存以“QueryConst.QUERY_ABSTRACTELEMENT_PROPERTY_KEY_PKEY”
 为key， 以实体主属性为value的Entity</div>
</td>
</tr>
<tr id="i24" class="altColor">
<td class="colFirst"><code>Entity</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEntityByFullName-java.lang.String-java.lang.String-">getEntityByFullName</a></span>(java.lang.String&nbsp;entityFullName,
                   java.lang.String&nbsp;tenantID)</code>
<div class="block">根据Entity全名查询相应的Entity</div>
</td>
</tr>
<tr id="i25" class="rowColor">
<td class="colFirst"><code>Entity</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEntityByID-java.lang.String-java.lang.String-">getEntityByID</a></span>(java.lang.String&nbsp;id,
             java.lang.String&nbsp;tenantID)</code>
<div class="block">通过ID查询Entity对方法</div>
</td>
</tr>
<tr id="i26" class="altColor">
<td class="colFirst"><code>Entity</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEntityByName-java.lang.String-java.lang.String-java.lang.String-">getEntityByName</a></span>(java.lang.String&nbsp;namespace,
               java.lang.String&nbsp;entityClassName,
               java.lang.String&nbsp;tenantID)</code>
<div class="block">根据Entity的Name查询相应的Entity</div>
</td>
</tr>
<tr id="i27" class="rowColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/Enumerate.html" title="com.yonyou.metadata.spi中的类">Enumerate</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEnumerate-java.lang.String-java.lang.String-java.lang.String-">getEnumerate</a></span>(java.lang.String&nbsp;namespace,
            java.lang.String&nbsp;name,
            java.lang.String&nbsp;tenantID)</code>
<div class="block">通过“命名空间”、枚举的全限定名，获取其所有的枚举对象。</div>
</td>
</tr>
<tr id="i28" class="altColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/Enumerate.html" title="com.yonyou.metadata.spi中的类">Enumerate</a>[]</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEnumerates-java.lang.String:A-java.lang.String-">getEnumerates</a></span>(java.lang.String[]&nbsp;enumerateIDs,
             java.lang.String&nbsp;tenantID)</code>&nbsp;</td>
</tr>
<tr id="i29" class="rowColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/EnumItem.html" title="com.yonyou.metadata.spi中的类">EnumItem</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEnumItemByName-java.lang.String-java.lang.String-java.lang.String-java.lang.String-">getEnumItemByName</a></span>(java.lang.String&nbsp;namespace,
                 java.lang.String&nbsp;enumeratename,
                 java.lang.String&nbsp;name,
                 java.lang.String&nbsp;tenantID)</code>
<div class="block">通过“命名空间”、枚举的类名、枚举值的名，获取其所对应的枚举值对象。</div>
</td>
</tr>
<tr id="i30" class="altColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/EnumItem.html" title="com.yonyou.metadata.spi中的类">EnumItem</a></code></td>
<td class="colLast"><code><span class="memberNameLink">getEnumItemByValue</a></span>(java.lang.String&nbsp;namespace,
                  java.lang.String&nbsp;name,
                  java.lang.String&nbsp;value,
                  java.lang.String&nbsp;tenantID)</code>
<div class="block">通过“命名空间”、枚举名、枚举值的value，获取其所对应的枚举值对象。</div>
</td>
</tr>
<tr id="i31" class="rowColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/EnumItem.html" title="com.yonyou.metadata.spi中的类">EnumItem</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEnumItems-java.lang.String-java.lang.String-java.lang.String-">getEnumItems</a></span>(java.lang.String&nbsp;namespace,
            java.lang.String&nbsp;name,
            java.lang.String&nbsp;tenantID)</code>
<div class="block">通过“命名空间”、枚举的全限定名，获取其所有的枚举值对象。</div>
</td>
</tr>
<tr id="i32" class="altColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/EnumItem.html" title="com.yonyou.metadata.spi中的类">EnumItem</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEnumItemsByCompID-java.lang.String-java.lang.String-java.lang.String-">getEnumItemsByCompID</a></span>(java.lang.String&nbsp;componentID,
                    java.lang.String&nbsp;name,
                    java.lang.String&nbsp;tenantID)</code>
<div class="block">此方法用于EnumitemList内部用于实现getEnumList的方法（由于从Enumerate中只能够获取componentID属性）</div>
</td>
</tr>
<tr id="i33" class="rowColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/EnumItem.html" title="com.yonyou.metadata.spi中的类">EnumItem</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEnumItemsByID-java.lang.String-java.lang.String-">getEnumItemsByID</a></span>(java.lang.String&nbsp;enumerateid,
                java.lang.String&nbsp;tenantID)</code>&nbsp;</td>
</tr>
<tr id="i34" class="altColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/EnumItem.html" title="com.yonyou.metadata.spi中的类">EnumItem</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getEnumItemsByValues-java.lang.String-java.lang.String-java.lang.String-java.lang.String...-">getEnumItemsByValues</a></span>(java.lang.String&nbsp;namespace,
                    java.lang.String&nbsp;name,
                    java.lang.String&nbsp;tenantID,
                    java.lang.String...&nbsp;values)</code>
<div class="block">通过“命名空间”、枚举的全限定名、多个枚举值的value，获取其所对应的枚举值对象。</div>
</td>
</tr>
<tr id="i35" class="rowColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getExtendAttribute-java.lang.String-java.lang.String-java.lang.String-">getExtendAttribute</a></span>(java.lang.String&nbsp;namespace,
                  java.lang.String&nbsp;name,
                  java.lang.String&nbsp;tenantID)</code>
<div class="block">通过类名查询其所有的扩展属性方法</div>
</td>
</tr>
<tr id="i36" class="altColor">
<td class="colFirst"><code>int</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getExtendAttributeCount-java.lang.String-java.lang.String-">getExtendAttributeCount</a></span>(java.lang.String&nbsp;name,
                       java.lang.String&nbsp;tenantID)</code>
<div class="block">通过类名查询其所有的扩展属性方法的数量，</div>
</td>
</tr>
<tr id="i37" class="rowColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getImplementsAttribute-java.lang.String-java.lang.String-java.lang.String-java.lang.String-">getImplementsAttribute</a></span>(java.lang.String&nbsp;interfaceName,
                      java.lang.String&nbsp;interfaceAttibuteName,
                      java.lang.String&nbsp;name,
                      java.lang.String&nbsp;tenantID)</code>
<div class="block">通过业务接口的全限定名和业务接口的属性名以及实现类的全限定名取得实现类的属性名。</div>
</td>
</tr>
<tr id="i38" class="altColor">
<td class="colFirst"><code>java.util.Map&lt;java.lang.String,java.util.Map&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>,<a hredf="../../../../../com/yonyou/metadata/spi/Attribute.html" title="com.yonyou.metadata.spi中的类">Attribute</a>&gt;&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getImplementsAttributes-java.lang.String-java.lang.String-java.lang.String-">getImplementsAttributes</a></span>(java.lang.String&nbsp;namespace,
                       java.lang.String&nbsp;name,
                       java.lang.String&nbsp;tenantID)</code>
<div class="block">通过实现类的全限定名获取其所实现的所有业务接口在本类中对应的属性列表。</div>
</td>
</tr>
<tr id="i39" class="rowColor">
<td class="colFirst"><code>java.util.Map&lt;java.lang.String,java.lang.Object&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getImplementsAttributesByFK-java.lang.String-java.lang.String-java.lang.String-">getImplementsAttributesByFK</a></span>(java.lang.String&nbsp;namespace,
                           java.lang.String&nbsp;name,
                           java.lang.String&nbsp;tenantID)</code>
<div class="block"><span class="deprecatedLabel">已过时。</span>&nbsp;</div>
</td>
</tr>
<tr id="i40" class="altColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/event/MetadataChangeListener.html" title="com.yonyou.metadata.spi.event中的接口">MetadataChangeListener</a>[]</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getMetadataChangeListeners--">getMetadataChangeListeners</a></span>()</code>&nbsp;</td>
</tr>
<tr id="i41" class="rowColor">
<td class="colFirst"><code>java.lang.String</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getNameSpaceByCompID-java.lang.String-java.lang.String-">getNameSpaceByCompID</a></span>(java.lang.String&nbsp;id,
                    java.lang.String&nbsp;tenantID)</code>&nbsp;</td>
</tr>
<tr id="i42" class="altColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/Relation.html" title="com.yonyou.metadata.spi中的类">Relation</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getRelation-java.lang.String-java.lang.String-java.lang.String-java.lang.String-">getRelation</a></span>(java.lang.String&nbsp;namespace,
           java.lang.String&nbsp;startName,
           java.lang.String&nbsp;endName,
           java.lang.String&nbsp;tenantID)</code>
<div class="block">通过两个实体的Name，查询其关系，针对主子关系，startName是子实体名，endName是主实体名，针对REF关系，startName是主实体名，endName是子实体名。</div>
</td>
</tr>
<tr id="i43" class="rowColor">
<td class="colFirst"><code><a hredf="../../../../../com/yonyou/metadata/spi/Relation.html" title="com.yonyou.metadata.spi中的类">Relation</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getRelationComponetId-java.lang.String-java.lang.String-java.lang.String-java.lang.String-">getRelationComponetId</a></span>(java.lang.String&nbsp;componentID,
                     java.lang.String&nbsp;startName,
                     java.lang.String&nbsp;endName,
                     java.lang.String&nbsp;tenantID)</code>&nbsp;</td>
</tr>
<tr id="i44" class="altColor">
<td class="colFirst"><code>java.util.List&lt;<a hredf="../../../../../com/yonyou/metadata/spi/Relation.html" title="com.yonyou.metadata.spi中的类">Relation</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#getRelations-java.lang.String-java.lang.String-java.lang.String-java.lang.String-">getRelations</a></span>(java.lang.String&nbsp;namespace,
            java.lang.String&nbsp;startName,
            java.lang.String&nbsp;endName,
            java.lang.String&nbsp;tenantID)</code>
<div class="block">通过两个实体的Name，查询其关系，针对主子关系，startName是子实体名，endName是主实体名，针对REF关系，startName是主实体名，endName是子实体名。</div>
</td>
</tr>
<tr id="i45" class="rowColor">
<td class="colFirst"><code>boolean</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#isImplementBizInterface-java.lang.String-java.lang.String-java.lang.String-">isImplementBizInterface</a></span>(java.lang.String&nbsp;entityID,
                       java.lang.String&nbsp;interfacename,
                       java.lang.String&nbsp;tenantID)</code>
<div class="block">查询entityID对应的实体是否实现的itfName对应的业务接口</div>
</td>
</tr>
<tr id="i46" class="altColor">
<td class="colFirst"><code>void</code></td>
<td class="colLast"><code><span class="memberNameLink"><a hredf="../../../../../com/yonyou/metadata/spi/service/MetadataService.html#removeMetadataChangeListener-com.yonyou.metadata.spi.event.MetadataChangeListener-">removeMetadataChangeListener</a></span>(<a hredf="../../../../../com/yonyou/metadata/spi/event/MetadataChangeListener.html" title="com.yonyou.metadata.spi.event中的接口">MetadataChangeListener</a>&nbsp;l)</code>&nbsp;</td>
</tr>
</table>