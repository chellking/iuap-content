# 分布式锁组件 #

## 业务需求 ##
很多业务上都有对同一个资源并发进行读写访问的场景，又要保证单次访问的正确性和多次访问的一致性，这就要求对资源进行加锁。同时在服务部署可以水平伸缩的情况下，无法做到在单JVM中的加锁控制，所以需要有一个分布式的系统来统一进行加解锁处理。

## 解决方案 ##
iuap的分布式锁组件是利用Zookeeper的强一致特性，通过Zookeeper来提供分布式锁的能力，解决了在多JVM中对共享资源加锁解锁的需求，并通过集群方式保证整个系统的高可用。本组件实现了对共享资源的互斥加解锁，在同一线程内的重复加锁，以及在业务结束后通过拦截触发的自动解锁功能等。

## 功能说明 ##
1.	支持对共享资源的互斥加解锁；
2.	支持线程内可重复加锁功能；
3.	支持批量加解锁功能；
4.	支持线程结束自动解锁功能。

锁的逻辑实现的流程图如下：

![](./images/iuap_lock_flow.jpg)


# 使用说明 #

## 配置和使用方式 ##
**1:在属性文件中，配置分布式锁组件使用的Zookeeper的连接url和方式，支持单机方式、集群方式**
	
	#配置锁组件连接zookeeper的url
	zklock.url=127.0.0.1:2181
	#zookeeper集群方式配置示例
	#zklock.url=20.12.6.58:2181,20.12.6.59:2181,20.12.6.60:2181

	#锁存活最大时间，单位秒，如果不配置，不强制删除，如果配置，加锁失败且已有的锁存活时间大于此值，强制删除
	zklock.maxlocktime=3600

**2:Spring的配置文件中，声明iuap-lock中连接池建立的配置信息的bean,根据项目需要调整参数值**

在web.xml中引入iuap-lock对应的spring配置文件，也可以将内部的配置合并到其他的spring配置文件中。

	<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
	  	    classpath*:/applicationContext-lock.xml
	    </param-value>
    </context-param>

主要是配置了连接池的信息，具体配置如下：

	<bean id="zkPoolConfig" class="org.apache.commons.pool2.impl.GenericObjectPoolConfig">
		<property name="maxTotal" value="100"/>
		<property name="maxIdle" value="20"/>
		<property name="maxWaitMillis" value="60000"/>
		
		<property name="timeBetweenEvictionRunsMillis" value="60000"/>
		<property name="numTestsPerEvictionRun" value="-1"/>
		<property name="minEvictableIdleTimeMillis" value="30000"/>
	</bean>

**3:工程中引入对iuap-lock组件的依赖**

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-lock</artifactId>
		<version>${iuap.modules.version}</version>
	</dependency>
${iuap.modules.version} 为平台在maven私服上发布的组件的version。


**4:初始化连接池**

在应用启动时，需要初始化连接池，可以在web.xml中定义listener，在listener中实现初始化，示例如下：

	public void contextInitialized(ServletContextEvent event) {
		WebApplicationContext wac = WebApplicationContextUtils.getWebApplicationContext(event.getServletContext());
		ContextHolder.setContext(wac);
		GenericObjectPoolConfig config = (GenericObjectPoolConfig)ContextHolder.getContext().getBean("zkPoolConfig");
		ZkPool.initPool(config);
	}


**5:代码中调用组件提供的API，操作分布式锁的加锁和解锁，完成业务**

	@Test
	public void testDisLockCodeExample() throws InterruptedException {
		String lockpath = "iuapdislockpath_test";
		boolean islocked = false;
		try {
			islocked = DistributedLock.lock(lockpath);
			if(islocked){
				System.out.println("加锁2成功，业务处理!");
			}
		} catch (Exception e) {
			System.out.println("加锁2操作异常!");
		} finally {
			if(islocked){
				DistributedLock.unlock(lockpath);
				System.out.println("解锁2成功!");
			} 
		}
	}

**6：Web项目中，可以配置过滤器方式，防止忘记解锁（不强制）**

	<filter>
	    <filter-name>lockfilter</filter-name>
	    <filter-class>com.yonyou.iuap.lock.filter.DistributedLockFilter</filter-class>
	</filter>
	<filter-mapping>
	    <filter-name>lockfilter</filter-name>
	    <url-pattern>/*</url-pattern>
	</filter-mapping>

	注意：url-pattern要按照自己加锁业务的请求路径进行映射，不要使用/*，以免拦截范围过大。

**7:更多API操作和配置方式，请参考分布式锁对应的示例工程(DevTool/examples/example_iuap_lock)**

## API接口 ##

###加锁操作###

- 功能描述

对敏感、关键的业务数据或资源进行加锁操作，加锁后可以进行业务操作，过程中不会有其他请求加锁成功。

- 调用方式

	boolean islocked = DistributedLock.lock(lockpath);

- 参数说明

<table style="border-collapse:collapse">
  <tbody><tr>
    <th>参数</th>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>lockpath</td>
    <td>String</td>
    <td>需要锁定资源的唯一标识</td>
  </tr>
</tbody></table>

- 返回值说明
	
<table style="border-collapse:collapse">
  <tbody><tr>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>boolean</td>
    <td>加锁成功或者失败的标识</td>
  </tr>
</tbody></table>

###解锁操作###

- 功能描述

对加锁后的资源进行解锁操作。

- 调用方式

	DistributedLock.unlock(lockpath);

- 参数说明

<table style="border-collapse:collapse">
  <tbody><tr>
    <th>参数</th>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>lockpath</td>
    <td>String</td>
    <td>需要解锁资源的唯一标识</td>
  </tr>
</tbody></table>

###批量加锁操作###

- 功能描述

对多个资源同时加锁，批量加锁要么全部加锁成功，要么全部加锁失败。

- 调用方式

	boolean islocked = DistributedLock.lock(locks);

- 参数说明

<table style="border-collapse:collapse">
  <tbody><tr>
    <th>参数</th>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>locks</td>
    <td>String[]</td>
    <td>需要锁定资源的唯一标识数组</td>
  </tr>
</tbody></table>

- 返回值说明
	
<table style="border-collapse:collapse">
  <tbody><tr>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>boolean</td>
    <td>批量加锁成功或者失败的标识</td>
  </tr>
</tbody></table>

###批量解锁操作###

- 功能描述

对加锁后的资源进行批量解锁操作。

- 调用方式

	DistributedLock.unlock(locks);

- 参数说明

<table style="border-collapse:collapse">
  <tbody><tr>
    <th>参数</th>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>locks</td>
    <td>String[]</td>
    <td>需要批量解锁资源的唯一标识数组</td>
  </tr>
</tbody>
</table>