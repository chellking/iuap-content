#分布式缓存组件概述#

## 业务需求 ##

为了提高业务应用的性能，避免对数据库资源的频繁访问，应用中常常引入缓存技术，将需要访问的资源或初次调用后的结果缓存起来，针对后续的相同访问直接返回缓存结果，以支持应用的高并发。

业务缓存的实现方式可以是将数据放置在JVM的堆内存中，也可以将数据放置在第三方中间件中,如MemCached、Redis。其中JVM内存缓存有两个弊端：一是大小受到JVM堆内存的限制，会引起频繁GC；二是集群部署时候，web应用间存在多份缓存，无法共享。第三方缓存中间件解决了JVM内存的这两个问题，但在操作中间件的缓存时，需要建立统一的连接并维护连接池，同时为了不绑定在某个特定的中间件上，还需要对多种中间件进行适配，这些都需要平台提供标准组件来进行处理。


## 解决方案 ##

Redis针对数据结构、持久化、过期策略、分布式集群、数据备份和灾难恢复等方面相对缓存中间件更有优势。iuap平台采用Redis作为缓存的中间件，并针对Redis提供统一的连接以及基本Java API封装，更利于方便业务开发简便快速的实现对业务数据的缓存操作，提高系统的运行效率。

iuap缓存组件还实现了对连接池的管理，并支持Redis以主从方式或者分片方式进行部署。分布式的架构保证了缓存服务的高可用性，在主从模式下支持主节点宕机后的自动切换，业务服务无感知。

## 功能说明 ##
1.	支持对缓存的读写操作以及过期时间设置；
2.	支持复杂类型数据的操作，如HashMap，Set，List；
3.	支持Redis主从和分片集群的不同配置；
4.	支持Redis的密码授权访问；
5.	支持Redis连接池配置管理；
6.	支持阿里云Redis数据库适配。


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖Jedis的2.6.0版本和iuap平台的一些基础组件如iuap-log，其对外提供的依赖方式如下：

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-cache</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 部署结构 ##

Redis本身支持多种语言的客户端来连接，iuap-cache组件利用java语言通过Jedis客户端进行连接，建立连接池并支持哨兵方式的url。

Redis中间件通常是配合哨兵进行集群部署，一主多从的部署结构和连接的示意图如下： 

<img src="/images/redis_sentinel.png"/>

# 使用说明 #

## 组件包说明 ##

iuap-cache组件利用jedis客户端，在springside提供的对jedis的封装的基础上，提供使用简易url的方式创建连接池、兼容直连和哨兵方式连接，模板类支持Redis的分片。

## 组件配置 ##

**1:在属性文件中，配置redis的连接url，根据业务不同的需要，可以配置成单机方式、集群方式、分片方式等**

	单机方式示例：
	redis.url=direct://localhost:6379?poolName=mypool&poolSize=100
	
	集群示例：
	redis.url=sentinel://20.12.6.57:26379,20.12.6.58:26379,20.12.6.59:26379?poolName=mypool&masterName=mymaster&poolSize=100
	
	分片示例：
	redis.shardedurl=direct://20.12.6.57:6379,20.12.6.58:6379,20.12.6.59:6379?poolName=mypool&masterName=mymaster&poolSize=100

**2:Spring的配置文件中，声明iuap-cache中的bean**

	<bean id="redisPool" class="com.yonyou.iuap.cache.redis.RedisPoolFactory" scope="prototype" factory-method="createJedisPool">
		<constructor-arg value="${redis.url}" />
	</bean>
	
	<bean id="jedisTemplate" class="org.springside.modules.nosql.redis.JedisTemplate">
		<constructor-arg ref="redisPool"></constructor-arg>
	</bean> 
	
    <bean id="cacheManager" class="com.yonyou.iuap.cache.CacheManager">
        <property name="jedisTemplate" ref="jedisTemplate"/>
    </bean>	

**3:工程中引入对iuap-cache组件的依赖**

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-cache</artifactId>
		<version>${iuap.modules.version}</version>
	</dependency>	

${iuap.modules.version}为在pom.xml中定义的引用iuap-cache的版本。

**4:代码中调用组件提供的API，操作分布式缓存**

	/**
	 * 普通存取、删除测试,get,set,remove,exists
	 * 
	 * @throws Exception
	 */
	@Test
	public void testSimpleCache() throws Exception {
		String key = "testKey";
		cacheManager.set(key, "test");
		
		Assert.isTrue(cacheManager.exists(key));
		
		String cacheValue = (String)cacheManager.get(key);
		System.out.println(cacheValue);
		
		cacheManager.removeCache(key);
		
		Assert.isNull(cacheManager.get(key));
	}

**5:更多API操作和配置方式，请参考缓存对应的示例工程(DevTool/examples/example\_iuap\_cache)**

## 工程样例 ##

开发工具包DevTool中携带了对分布式缓存组件的示例工程，位置位于DevTool/examples/example_iuap_cache下，在iuap Studio中导入已有的Maven工程，可以将示例工程导入到工作区。示例工程中有较为完整的对iuap-cache组件的使用示例代码。
<img src="/images/cache_example.jpg"/>

## 开发步骤 ##

- 参考上述配置项，配置属性文件中的redis连接串，引入缓存对应的spring配置文件applicationContext-cache.xml，如果是web应用，修改web.xml如下：

		<context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>
				classpath*:/applicationContext.xml,
				classpath*:/applicationContext-cache.xml,
				classpath*:/applicationContext-persistence.xml,
				classpath*:/applicationContext-shiro.xml
			</param-value>
		</context-param>
		<context-param>
			<param-name>spring-mvc-servlet-name</param-name>
			<param-value>springServlet</param-value>
		</context-param>
		<listener>
			<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
		</listener>

- 属性文件中配置的redis应该是服务器的redis地址，如果本机调试，可以启动DevTool中默认携带的redis，启动脚本位于DevTool->bin->startRedis.bat,双击运行，效果如下：
<img style="margin-left:25px;" src="/images/cache_redis.jpg"/>

- 注入在applicationContext-cache.xml中声明的bean cacheManager，如果应用中只有一个此类型的bean，则可以使用@Autowired注入，如果存在多个，则按照名称注入。业务开发时候，可以为不同的业务模块声明不同的cache区域，注册多份CacheManager和redisPool即可。

- iuap-cache组件默认使用的是java基础的序列化和反序列化方式，如果项目上需要替换更高性能的序列化方式，在声明CacheManager时候注入Serializer即可。

	    public void setSerializer(Serializer serializer) {
	        this.serializer = serializer;
	    }


## 常用接口 ##

- CacheManager

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
			<td>set(final String key, final T value)</td>
			<td>String key（缓存key）, T value（缓存对象，允许序列化）</td>
			<td>void</td>
			<td>放置键值对对应的缓存</td>
		</tr>
		<tr>
			<td>setex(final String key, final T value，final int timeout)</td>
			<td>String key（缓存key）, T value（缓存对象，允许序列化），int timeout（缓存的有效期）</td>
			<td>void</td>
			<td>放置键值对对应的缓存并设置缓存有效期，单位为秒</td>
		</tr>
		<tr>
			<td>expire(final String key, final int timeout)</td>
			<td>final String key（缓存key）, final int timeout（缓存的有效期）</td>
			<td>void</td>
			<td>修改缓存信息的有效期</td>
		</tr>
		<tr>
			<td>exists(final String key)</td>
			<td>final String key（缓存key）</td>
			<td>Boolean（是否存在的标志）</td>
			<td>判断键值为key的缓存是否存在</td>
		</tr>
		<tr>
			<td>get(final String key)</td>
			<td>final String key（缓存key）</td>
			<td>T extends Serializable（声明的返回类型对象）</td>
			<td>获取对应键值的缓存对象</td>
		</tr>
		<tr>
			<td>hget(final String key, final String fieldName)</td>
			<td>final String key（缓存Map的key），final String fieldName（Map下某个属性的key）</td>
			<td>T extends Serializable（声明的返回类型对象）</td>
			<td>获取HashMap中对应的子key的缓存值</td>
		</tr>
		<tr>
			<td>hmget(final String key, final String... fieldName)</td>
			<td>final String key（缓存Map的key）, final String... fieldName（Map下若干属性的key）</td>
			<td>List</td>
			<td>获取HashMap中对应的多个子key的缓存值集合</td>
		</tr>
		<tr>
			<td>hexists(final String key, final String field)</td>
			<td>final String key（缓存Map的key），final String field（Map下某个属性的key）</td>
			<td>Boolean（是否存在的标志）</td>
			<td>判断HashMap中对应的key是否存在</td>
		</tr>
		<tr>
			<td>hset(final String key, final String fieldName, final T value)</td>
			<td>final String key（缓存Map的key），final String fieldName（Map下某个属性的key），final T value（要放置的缓存对象）</td>
			<td>void</td>
			<td>放置HashMap中的属性和值</td>
		</tr>
		<tr>
			<td>removeCache(String key)</td>
			<td>String key（缓存的key）</td>
			<td>void</td>
			<td>删除key对应的缓存信息</td>
		</tr>
		<tr>
			<td>hdel(String key, String field)</td>
			<td>String key（缓存Map的key），String field（Map下某个属性的key）</td>
			<td>void</td>
			<td>删除Map下键值为filed的属性</td>
		</tr>
	</tbody>
</table>





## redis cluster方式使用
redis官方在redis3.0以后的版本提供了 redis-cluster的方式

组件采用Maven进行编译和打包发布，依赖Jedis的2.9.0版本.
 redis支持的cluster特性：

　　1):节点自动发现(自动发现集群中的其他节点)

　　2):slave->master 选举,集群容错

　　3):Hot resharding:在线分片

　　4):集群管理:cluster xxx

　　5):基于配置(nodes-port.conf)的集群管理

　　6):ASK 转向/MOVED 转向机制.
             缓存服务器获取一个Key要经过二次定位.

### 架构细节:

(1)所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽.

(2)节点的fail是通过集群中超过半数的节点检测失效时才生效.

(3)客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可

(4)redis-cluster把所有的物理节点映射到[0-16383]slot上,cluster 负责维护node<->slot<->value
 

### 程序中使用cluster方式


1、修改属性文件，添加以下属性

    #redis3.+以后支持的 cluster操作配置
    genericPool.maxWaitMillis=-1
    genericPool.maxTotal=100
    genericPool.minIdle=8
    genericPool.maxIdle=100
    rediscluster.session.url=192.168.22.129:7000,192.168.22.129:7001,192.168.22.129:7002,192.168.22.129:7003,192.168.22.129:7004,192.168.22.129:7005?timeout=20000&maxRedirections=6
    rediscluster.url=192.168.22.129:7000,192.168.22.129:7001,192.168.22.129:7002,192.168.22.129:7003,192.168.22.129:7004,192.168.22.129:7005?timeout=20000&maxRedirections=6
    
    #session放入到那种redis缓存里面， normal为原来的形式， cluster为放入到 redisCluster里面
    redis.seesion.type=cluster
    
    

修改 applicationContext-shiro.xml 文件

     <!-- session jedisCluster -->
   	<bean name="genericObjectPoolConfig" class="org.apache.commons.pool2.impl.GenericObjectPoolConfig" >
			<property name="maxWaitMillis" value="${genericPool.maxWaitMillis}" />
			<property name="maxTotal" value="${genericPool.maxTotal}" />
			<property name="minIdle" value="${genericPool.minIdle}" />
			<property name="maxIdle" value="${genericPool.maxIdle}" />
	</bean>
 
	<bean id="sessionJedisCluster" class="com.yonyou.iuap.cache.redis.JedisClusterSetFactory">
		<property name="url" value="${rediscluster.session.url}" />    
		<property name="genericObjectPoolConfig" ref="genericObjectPoolConfig" />
	</bean>
	<bean id="sessionClusterMgr" class="com.yonyou.iuap.auth.session.SessionClusterManager">
		<property name="jedisCluster" ref="sessionJedisCluster"  />
	</bean>
	
	
	
java代码调用
	
    @Autowired
	private    SessionClusterManager sessionMgr;
	
	@Autowired
	private CacheClusterManager cacheManager;
	
	@Autowired()
	protected ITokenProcessor webTokenProcessor;

 
    @Test
	public void mockLogin() {
		for (int i = 0; i < 100; i++) {
			String userId = "testuser_" + i;
			TokenParameter tokenparam = new TokenParameter();
			tokenparam.setUserid(userId);
			tokenparam.setLogints(String.valueOf(System.currentTimeMillis()));
			sessionMgr.registOnlineSession(userId, webTokenProcessor.generateToken(tokenparam), webTokenProcessor);
		}
	}



### redis集成spring方法级缓存


## 方法级缓存使用
 
####  1 、配置文件中添加配置

    <cache:annotation-driven cache-manager="simpleCacheManager"/>    
	
	<!--   name: 注解需要的名字 （必填）
	       cacheManager  缓存操作 （必填）
	       expireTime 过期时间 ，单位秒  （可空） -->
	<bean id="simpleCacheManager" class="org.springframework.cache.support.SimpleCacheManager">
	    <property name="caches"> 
	    	<set>
	    		<bean class="com.yonyou.iuap.cache.spring.SimpleCache" >
	    			<property name="name"  value="redisCache"></property>   
	    			<property name="cacheManager" ref="cacheManager"></property>
	    			<property name="expireTime" value="30"></property>
	    		</bean>
	    		<bean class="com.yonyou.iuap.cache.spring.SimpleCache" >
	    			<property name="name"  value="redisCacheTwo"></property>   
	    			<property name="cacheManager" ref="cacheManager"></property>
	    		</bean>
	    	 
	    	</set>
	    </property>
	</bean>
	
	
	
#### 2、 在需要写入缓存的方法上加入注解（java代码如下）cacheService.java
	
    @Cacheable(value = "redisCache", key = "#key")  
	public Object getFromDB(String key, Object obj ){
		
		System.out.println("exeute getFromDb ....");
		return  obj ;
	}
	
	
	
#### 注解说明：
  
 @Cacheable(value=”testcache”,key="#key" , condition=”#userName.length()>2”)
 
> @Cacheable 主要的参数说明
>
>   value : 缓存的名称，在 spring 配置文件中定义，必须指定至少一个。
>   
>   key  : 缓存的 key，可以为空，如果指定要按照 SpEL 表达式编写，如果不指定，则缺省按照方法的所有参数进行组合。
>   
>   condition :  缓存的条件，可以为空，使用 SpEL 编写，返回 true 或者 false，只有为 true 才进行缓存
 
  
  
   @CachePut(value=”testcache”,condition=”#userName.length()>2”)
   
>  和 @Cacheable 不同的是，它每次都会触发真实方法的调用
      
 @CachEvict(value=”testcache”,key=”#userName”)           
>  @CachEvict   主要针对方法配置，能够根据一定的条件对缓存进行清空。

>   value  缓存的名称

>   key 缓存的 key，可以为空，如果指定要按照 SpEL 表达式编写，如果不指定，则缺省按照方法的所有参数进行组合
>         例如：
>         @CachEvict(value=”testcache”,key=”#userName”)

>   condition     缓存的条件，可以为空，使用 SpEL 编写，返回 true 或者 false，只有为 true 才清空缓存。

>     例如： @CachEvict(value=”testcache”,condition=”#userName.length()>2”)

> allEntries   是否清空所有缓存内容，缺省为 false

> beforeInvocation   是否在方法执行前就清空，缺省为 false

  
	
	
#### 3、 在其他对象里面调用被注解的方法

    @Autowired
    private CacheService cacheService ;
    
    
	public void testRedisCacheSet(){
		Object value = cacheService.getFromDB("testSpringCache","fromDb_value") ;
		System.out.println("the value is:   " + value );
		Assert.notNull(value);
	}
	
#### 4、 注意 ：需要调用缓存的方法要外部调用才能生效，不能内部调用（即不能用 this 调用，否则会失效）