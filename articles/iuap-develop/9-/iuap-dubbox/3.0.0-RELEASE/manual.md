#Dubbox扩展组件#

##业务需求##
在构建大型业务系统时，为了提高系统整体的并发吞吐量，往往需要对系统按照相对独立的部分进行拆分，各部分以服务的方式向外提供，以便于针对不同的服务进行不同的部署和扩展。服务的提供方式可以是多样的，如REST服务、WebService、RPC分布式服务等。

分布式服务的出现解决了以往模块之间的紧耦合问题和单独部署扩展等问题，但在服务调用过程中如何传递上下文传递需要特殊的考虑。例如，在分布式服务调用时，希望将当前的用户登录信息、租户信息、线程号等传递到服务提供方，方便后续业务的日志输出、错误跟踪、信息持久化等，这就需要规范服务的上下文传递过程，保证统一性。

##解决方案##
Dubbo是一个被国内很多互联网公司广泛使用验证的开源分布式服务框架，Dubbox是在Dubbo的基础上，封装了Restful调用以及做了一些jar的版本升级。它解决了分布式服务中的服务自动注册发现和服务治理问题，使得服务可以动态扩容、缩容，服务也可以根据使用情况实现升降级。

iuap的Dubbox扩展组件适配Dubbo 2.8.4，基于Spring 4.0.5 重新编译，支持服务调用过程中的上下文信息传递。开发者可以在Dubbo服务声明时指定过滤器，配置传递的上下文信息。在使用过程中，服务消费者向服务提供者自动传递上下文，传递的信息包括当前上下文中的用户信息、线程信息等，还可以包括定制的一些信息。

##功能说明##

1.	支持分布式服务调用间的上下文信息传递；
2.	支持上下文信息的定制；
3.	适配Spring 4.0.5；


# 整体设计 #

## 依赖环境 ##
iuap-dubbox组件依赖dubbo 2.8.4版本，是基于spring 4.0.5 重新编译的，会随着iuap的开发工具包发布，内部依赖方式如下：

	<dependency>
	    <groupId>com.alibaba</groupId>
	    <artifactId>dubbo</artifactId>
	    <version>2.8.4.RELEASE</version>
	    <exclusions>
	    	<exclusion>
	    		<artifactId>log4j</artifactId>
	    		<groupId>log4j</groupId>
	    	</exclusion>
	    </exclusions>
	</dependency>
	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-generic</artifactId>
		<version>${iuap.modules.version}</version>
	</dependency>

iuap-dubbox组件依赖上述jar包，外部项目直接依赖iuap-dubbox和Zookeeper的相关客户端即可，示例如下：

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-dubbox</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>

	 <dependency>
	    <groupId>com.github.sgroschupf</groupId>
	    <artifactId>zkclient</artifactId>
	    <version>${zkclient.version}</version>
	    <exclusions>
	        <exclusion>
	            <artifactId>log4j</artifactId>
	            <groupId>log4j</groupId>
	        </exclusion>
	    </exclusions>
	</dependency>
	<dependency>
	    <groupId>org.apache.zookeeper</groupId>
	    <artifactId>zookeeper</artifactId>
	    <version>${zookeeper.version}</version>
	    <exclusions>
	        <exclusion>
	            <artifactId>slf4j-log4j12</artifactId>
	            <groupId>org.slf4j</groupId>
	        </exclusion>
	        <exclusion>
	            <artifactId>log4j</artifactId>
	            <groupId>log4j</groupId>
	        </exclusion>
	    </exclusions>
	</dependency>

${iuap.modules.version}是在pom.xml中定义的引用组件的版本。Zookeeper推荐版本为3.4.6，示例中使用的zkclient版本为0.1。

## 功能结构 ##

**基本概念**

- 服务提供者

    暴露服务的服务提供方

- 服务消费者

    调用远程服务的服务消费方

- 注册中心  

    服务注册与发现的注册中心，我们使用zookeeper作为示例注册中心。
<img src="/images/dubbox.png"/>

iuap-dubbox在原有dubbo服务的基础上，以Filter的方式进行扩展，增加上下文业务信息和日志信息的传递。

# 使用说明 #

##配置方式##
**1:在属性文件中，配置zookeeper连接地址、协议名称、负载策略等**

	dubbo.application.name=dubbox_provider
	dubbo.registry.address=zookeeper://localhost:2181
	dubbo.monitor.protocol=registry
	dubbo.protocol.name=dubbo
	dubbo.protocol.port=20880
	dubbo.service.loadbalance=consistenthash
	dubbo.log4j.file=logs/dubbox_provider.log
	dubbo.log4j.level=DEBUG

**2:增加dubbox对应的provider、customer的Spring配置文件**
	

- 1）provider示例：applicationContext-dubbo-provider.xml

		<!-- dubbo 配置 -->
		<dubbo:application name="${dubbo.application.name}"/>
		<dubbo:registry address="${dubbo.registry.address}"/>
		<dubbo:protocol name="${dubbo.protocol.name}" port="${dubbo.protocol.port}"/>
	
		<dubbo:monitor protocol="registry"></dubbo:monitor>
		<!-- 生产者声明 -->
		<bean id="dubboxProvider" class="uap.web.dubbox.DubboxProviderImplTest"/>
		<dubbo:service interface="uap.web.dubbox.DubboxProviderItfTest" ref="dubboxProvider" version="0.1" retries="0" timeout="35000" loadbalance="${dubbo.service.loadbalance}" filter="logcontext"/>

- 2）consumer示例：applicationContext-dubbo-consumer.xml

	    <!-- dubbo 配置 -->
	    <dubbo:application name="${dubbo.application.name}"/>
	    <dubbo:registry address="${dubbo.registry.address}" timeout="15000"/>
	    <dubbo:consumer check="false"/>
	
	    <!-- 消费者声明 -->
	    <dubbo:reference id="dubboxConsumer" interface="uap.web.dubbox.DubboxProviderItfTest" version="0.1" filter="logcontext"/>

**3:在服务消费者的工程上注入bean，调用服务**

**4:其他详细配置和参数，请参考示例工程(DevTool/examples/example\_dubbox\_consumer和example\_dubbox\_provider)和官网文档**

##工程样例##

- 启动Zookeeper，在DevTool的bin目录下，启动Zookeeper作为注册中心

- 启动服务提供者，导入iuap-dubbox-example-provider，以单元测试方式启动即可

- 模拟消费者调用，导入iuap-dubbox-example-consumer，单元测试方式运行TestDubboxConsumer中的方法