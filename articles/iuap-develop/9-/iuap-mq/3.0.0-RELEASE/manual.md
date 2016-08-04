# 消息组件 #

## 业务需求 ##
业务系统中为了提高核心服务的并发性，需要对不是强关联的服务之间进行解耦，使用消息实现彼此异步化，保障核心能力。在应对突发高流量的场景时，也需要有异步消息的机制来实现对请求的削峰填谷，保障核心业务不受影响。

##解决方案##
iuap的消息组件提供基于消息队列RabbitMQ和阿里云 MNS消息服务的功能封装，使用统一的方式来实现点对点消息和发布订阅消息，同时实现了对消息队列的自动监听和消费者调用，使用镜像集群的方式来保证消息的可靠存储，使用ACK机制保证消息的可靠消息，上下文信息自动在生产者和消费者之间传递，开发者只需要关注业务逻辑，更方便实现异步业务。

##功能说明##

1.	支持点对点消息模型；
2.	支持发布订阅消息模型；
3.	同时支持RabbitMQ和阿里云MNS服务；
4.	支持上下文信息在队列生产者和消费者之间传递；


# 使用说明 #

## Maven依赖 ##

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-mq</artifactId>
		<version>${iuap.modules.version}</version>
	</dependency>

iuap.modules.version为在pom.xml定义的需要引用组件的版本。

## 配置属性文件 ##
**在application.properties属性文件中，配置连接信息，根据项目选择配置不同的消息连接方式**

	#mq
	mq.username=admin
	mq.password=admin
	mq.addresses=localhost:5672

	#mns
	mns.accountendpoint=http://账号.mns.cn-区域.aliyuncs.com/
	mns.accesskeyid=阿里云开发者账号
	mns.accesskeysecret=对应开发者账号的secret

## 调整spring配置文件 ##

**如果是RabbitMQ方式，配置消息生产者和消费者对应的spring配置文件，文件中定义消息队列、监听等信息**

消息生产者对应的关键bean声明如下，更详细的配置请参考示例工程.

	<!-- 连接服务配置  -->
	<rabbit:connection-factory id="connectionFactory" addresses="${mq.addresses}"  username="${mq.username}" password="${mq.password}" publisher-confirms="false"/>
         
	<rabbit:admin connection-factory="connectionFactory"/>
    
	<!-- queue 队列声明-->
	<rabbit:queue id="simple_queue" durable="true" auto-delete="false" exclusive="false" name="simple_queue"/>
	<!-- exchange queue binging key 绑定，作为点对点模式使用 -->
	<rabbit:direct-exchange name="iuap-direct-exchange" durable="true" auto-delete="false" id="iuap-direct-exchange">
		<rabbit:bindings>
			<rabbit:binding queue="simple_queue" key="simple_queue_key"/>
		</rabbit:bindings>
	</rabbit:direct-exchange>
	<!-- 通用RabbitMQ producer声明 -->
	<bean id="rabbitMQProducer" class="com.yonyou.iuap.mq.rabbit.RabbitMQProducer">
		<property name="rabbitTemplate" ref="rabbitTemplate"></property>
	</bean>
	
	<!-- 增加失败重试机制，发送失败之后，会尝试重发三次，重发间隔(ms)为 
	    第一次 initialInterval 
	    此后：initialInterval*multiplier > maxInterval ? maxInterval : initialInterval*multiplier。
	    配合集群使用的时候，当mq集群中一个down掉之后，重试机制尝试其他可用的mq。
	 -->
	<bean id="retryConnTemplate" class="org.springframework.retry.support.RetryTemplate">
	    <property name="backOffPolicy">
	        <bean class="org.springframework.retry.backoff.ExponentialBackOffPolicy">
			    <property name="initialInterval" value="500"/>
			    <property name="multiplier" value="10.0"/>
			    <property name="maxInterval" value="5000"/>
	        </bean>
	    </property>
    </bean>


消息消费者方对应的关键bean声明如下，更详细的配置请参考示例工程.

	<rabbit:listener-container connection-factory="connectionFactory">
		<rabbit:listener queues="simple_queue" ref="queueLitener"/>
	</rabbit:listener-container>
	
	<!-- queue litener  观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象-->
	<rabbit:listener-container connection-factory="connectionFactory" acknowledge="auto">
		<rabbit:listener queues="simple_queue" ref="queueLitener"/>
	</rabbit:listener-container>

	<bean id="queueLitener" class="com.yonyou.iuap.mq.rabbitmq.MqTestListener"></bean>

**如果是阿里云MNS方式，配置MNS所需的客户端和服务，listener中可以定制轮循取消息空闲时候的间隔时间**

	<bean id="mnsAccount" class="com.aliyun.mns.client.CloudAccount">
		<constructor-arg index="0">           
			<value>${mns.accesskeyid}</value>       
		</constructor-arg>       
		<constructor-arg index="1">           
			<value>${mns.accesskeysecret}</value>       
		</constructor-arg>
		<constructor-arg index="2">           
			<value>${mns.accountendpoint}</value>       
		</constructor-arg>		
	</bean>

	<bean id="mqService" class="com.yonyou.iuap.mq.mns.AliyunMnsService">
		<property name="mnsAccount" ref="mnsAccount"></property>
	</bean>

	<bean id="simpleListener" class="com.yonyou.iuap.mq.mns.MnsSimpleListener">
		<constructor-arg index="0">           
			<value>testali-mq-01</value>       
		</constructor-arg>       
		<constructor-arg index="1">           
			<ref bean="mnsAccount"/>    
		</constructor-arg>
		<constructor-arg index="2">           
			<value>5</value>       
		</constructor-arg>
	</bean>

##API调用##

###消息发送###

**业务代码中，在需要发送消息的服务中，引入消息发送的Service，调用发送消息的API**

	@Autowired
	private IMqService mqService;
	
	//rabbitmq
	@Test
	public void queueMessage1() throws Exception {
		String qName = "iuap-direct-exchange";
		String key = "simple_queue_key";
		String msg = "iuap mq msg test! ";
		
		mqService.sendMsg(qName, key, msg);
	}
	
	//mns
	@Test
	public void queueMessage2() throws Exception {
		String qName = "testali-mq-01";
		String key = null;
		String msg = "iuap mq msg test! ";
		
		mqService.sendMsg(qName, key, msg);
	}

###消息监听###
**如果是MNS的普通队列的监听，需要编写监听类继承AbstractMessageListener**

	 public class MnsSimpleListener extends AbstractMessageListener {

		public MnsSimpleListener(String queueName, CloudAccount mnsAccount, int waitSeconds) {
			super(queueName, mnsAccount, waitSeconds);
		}

		@Override
		public void onMessage(Message message) {
			System.out.println(message.getMessageBody());
		}

	}

**如果是RabbitMQ的普通队列的监听，需要编写监听类实现MessageListener**

	@Service
	public class MqTestListener implements MessageListener{

		private static Logger logger = LoggerFactory.getLogger(MqTestListener.class);
		
		@Override
		public void onMessage(Message message) {
			logger.info("MQ ======== mq MqTestListener :" + new String(message.getBody()));
		}

	}

###消息订阅###

**如果是MNS的topic的监听，需要编写REST服务，供MNS回调，注意MNS要求服务的路径只能映射一层**

	@Controller
	public class MnsTopicListener extends AbstracTopicListener{
		
	    private final Logger logger = LoggerFactory.getLogger(getClass());
	    
	    @RequestMapping(value = "/notifications", method = RequestMethod.POST)
	    public void receiveMnsTopic(HttpServletRequest request,HttpServletResponse response) {
		
		//获取消息
		String message = fetchMessage(request,response);
		if(StringUtils.isNotBlank(message)){
			onMessage(message);
		}
	    }

	        // 业务逻辑，拿到消息体后的处理
		private void onMessage(String message) {
			logger.info(message);
		}
	    
	}

**更加详细的配置和示例请参考示例工程(DevTool/examples/example_iuap_mq)和官网文档**

###上下文传递###
注意：如果需要消息发送和接收时传递上下文，需要引入iuap-saas-mq，引入方式类似。

#### Rabbitmq传递上下文信息 ####


- 消息发送配置

		<bean id="rabbitMqContextService" class="com.yonyou.iuap.mq.rabbit.RabbitMqContextService">
			<property name="rabbitTemplate" ref="rabbitTemplate"></property>
		</bean>
- 发送代码

		@Resource(name="rabbitMqContextService")
		IMqService mqService ;
	
		/** 发送普通rabbitmq  上下文消息传递*/
		@Test
		public void queueMessageContextTest() throws InterruptedException {
			long t1 = System.currentTimeMillis();
			int msgCount = 2 ;
			for(int i = 0; i < msgCount; i++){
				TestMq t = new TestMq("testmq" + i, "guyz" + i ) ;
				JsonMapper objectMapper = new JsonMapper();
				String msg = objectMapper.toJson(t) ;
				mqService.sendMsg("iuap-direct-exchange", "simple_queue_key" , msg);
			}
		}


- 消息接收配置

	接收方继承类 com.yonyou.iuap.mq.rabbit.consumer.MqSaasListener ，并覆写 handleMessage() 方法。
	
		public class MqSaasTestListener extends MqSaasListener{
			private static Logger logger = LoggerFactory.getLogger(MqSaasTestListener.class);
		
			@Override   //接收消息处理具体业务逻辑
			public void handleMessage(String message){
				System.out.println(   InvocationInfoProxy.getCallid()  );
				System.out.println(  InvocationInfoProxy.getLocale() ); 
			} 
	
		}

####阿里云MNS传递上下文信息 ####

- 普通队列发送配置

		<bean id="mqService" class="com.yonyou.iuap.mq.mns.AliyunMnsSaasService">
			<property name="mnsAccount" ref="mnsAccount"></property>
		</bean>
		
	
	发送代码
	
		@Resource(name="mqService") 
		IMqService mqService ;
		
		//发送普通queue 到ali云 
		@Test
		public void queueMessageContext() throws InterruptedException {
			 
			int msgCount = 4 ;
			for (int i = 0; i < msgCount; i++) {
				TestMq t = new TestMq("testmq" + i, "guyz" + i) ;
				JsonMapper objectMapper = new JsonMapper();
				String msg = objectMapper.toJson(t) ;	 
				mqService.sendMsg("testali-mq-01", null, msg);
			}
		}
	

- 接收普通queue消息

 	接收阿里云的普通queue消息是通过主动去阿里获取消息的，需要实现MnsSaasListener类

	配置：
	
		<bean id="simpleListener" class="com.yonyou.iuap.mq.mns.MnsSimpleListener">
			<constructor-arg index="0">           
				<value>testali-mq-01</value>       
			</constructor-arg>       
			<constructor-arg index="1">           
				<ref bean="mnsAccount"/>    
			</constructor-arg>
			<constructor-arg index="2">           
				<value>5</value>       
			</constructor-arg>
		</bean>
	
	接收代码样例
	
	
		public class MnsSimpleListener extends MnsSaasListener {	
			public MnsSimpleListener(String queueName, CloudAccount mnsAccount, int waitSeconds) {
				super(queueName, mnsAccount, waitSeconds);
			} 
			public void handleMessage(String message){//根据业务需要处理
				System.out.println(   message );
				System.out.println(" ====InvocationInfoProxy.getCallid()  " + InvocationInfoProxy.getCallid() );
			}
		
			public void afterHandleMessage(){
				InvocationInfoProxy.reset() ; 
			}
		
		}


- 阿里云MNS topic类型消息发送

		<bean id="mqService" class="com.yonyou.iuap.mq.mns.AliyunMnsSaasService">
			<property name="mnsAccount" ref="mnsAccount"></property>
		</bean>


	java代码：
	
		@Resource(name="mqService") 
		IMqService mqService ;
		//发送给阿里云 topic ，包含上下文信息
		@Test
		public void testTopicMns() throws Exception {
			String topicName = "test-topic-01";
			String msg = "topic context message test! ";
			mqService.publishMsg(topicName, msg);
			System.out.println("topic方式发送消息完成"   );
		}
- MNS topic 消息订阅

	需要编写REST服务，供MNS回调，注意MNS要求服务的路径只能映射一层,如果为/appname/notifications,出现两个/则不合法


	要接收上下文信息需要实现 MnsTopicSaasListener
		@Controller
		public class MnsTopicSaasTestListener extends MnsTopicSaasListener{
		
			private final Logger logger = LoggerFactory.getLogger(getClass());
		    
		    @RequestMapping(value = "/notifications", method = RequestMethod.POST)
		    public void receiveMnsTopic(HttpServletRequest request,HttpServletResponse response) {
		    	super.receiveMnsTopic(request, response);
		    	
		    }
		
		    // 业务逻辑，拿到消息体后的处理
			@Override
			public void onMessage(String message) {
				//TODO  业务逻辑
				System.out.println("get  topic message :" + message );
				System.out.println(" ====InvocationInfoProxy.getCallid()  " + InvocationInfoProxy.getCallid() );
			}
		}