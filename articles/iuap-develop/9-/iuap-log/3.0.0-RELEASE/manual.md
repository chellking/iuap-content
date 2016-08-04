# 日志组件 #

## 业务需求 ##
在系统运行中，都需要记录运行日志，通过不同的配置指定不同内容和不同级别的日志输出到不同的存储，进一步利用日志记录的信息为系统进行监控排错、定位问题和优化性能。

##解决方案##
iuap后台日志中，统一使用slf4j+logback的方式进行日志的记录和输出，通过slf4j对各个日志组件的桥接器，适配不同日志件对日志的输出。
同时，日志组件采用slf4j和MDC的结合，可以在日志中记录如用户编号，线程调用唯一标识等指定信息，方便问题的追踪，为运维管理中如ELK方案追踪日志信息提供线索。

## 功能说明 ##
1.	支持适配不同日志组件对日志的输出；
2.	支持日志不同输出级别的设置；
3.	支持包、类不同粒度的日志输出设置；
4.	结合MDC，支持日志中输出字段的自定义；

#使用说明#

## 配置logback.xml ##
**在工程的classpath中加入logback的日志配置文件logback.xml,详细配置请参考示例工程**

日志的配置中，可以配置日志的输出级别，输出格式，自定义输出变量等，还可以指定具体包下的日志输出级别。

**示例配置文件如下**

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
		<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
			<encoder>
				<pattern>%date{HH:mm:ss.SSS} [%X{call_thread_id}] [%X{current_user_name}] [%thread] %-5level %logger{36} - %msg%n</pattern>
			</encoder>
		</appender>
	
		<appender name="rollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
			<file>/tmp/logs/iuap-logexample.log</file>
			<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
				<fileNamePattern>/tmp/logs/iuap-logexample.%d{yyyy-MM-dd}.log</fileNamePattern>
			</rollingPolicy>
			<encoder>
				<pattern>%date{HH:mm:ss.SSS} [%X{call_thread_id}] [%X{current_user_name}] [%thread] %-5level %logger{36} - %msg%n</pattern>
			</encoder>
		</appender>
	
		<!-- 可以定制某些包下的日志输出级别 -->
		<logger name="com.yonyou.iuap" level="DEBUG" />
	
		<!--log4jdbc是否支持 -->
		<logger name="jdbc.connection" level="OFF" />
		<logger name="jdbc.audit" level="OFF" />
		<logger name="jdbc.resultset" level="OFF" />
		<logger name="jdbc.sqlonly" level="OFF" />
		<logger name="jdbc.sqltiming" level="OFF" />
	
		<root level="DEBUG">
			<appender-ref ref="console" />
			<appender-ref ref="rollingFile" />
		</root>
	</configuration>

##配置组件的Maven依赖##

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-log</artifactId>
		<version>${iuap.modules.version}</version>
	</dependency>

iuap.modules.version为在pom.xml定义的需要引用组件的版本。

注意：iuap-log组件默认引入对slf4j、logback和各种桥接包的依赖，内部依赖不需要再次引入，依赖如下：

	<dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- 代码直接调用log4j会被桥接到slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!-- 代码直接调用common-logging会被桥接到slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!-- 代码直接调用java.util.logging会被桥接到slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jul-to-slf4j</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>com.googlecode.log4jdbc</groupId>
            <artifactId>log4jdbc</artifactId>
            <version>${log4jdbc.version}</version>
            <scope>runtime</scope>
        </dependency>
	</dependencies>

**调用方式**

**在代码中声明logger**

	private static final Logger logger = LoggerFactory.getLogger(LogTest.class);

**在业务代码中，调用logger的API输出各种级别的日志信息**

	/**
	 * 多参数示例
	 */
	@Test
	public void testMultiParamsLog() {
		logger.info("this is a {} log demo! level is：{}", "multi params", "info");
	}


**更多API操作和配置方式，请参考对应的示例工程(DevTool/examples/example_iuap_log)**

##常用接口##

### 输出简单日志信息 ###

	/**
	 * 简单示例
	 */
	@Test
	public void testSimpleLog() {
		
		logger.info("this is a simple log demo!");
	}
	

### 参数化输出日志 ###

	/**
	 * 参数化示例
	 */
	@Test
	public void testParamsLog() {
		logger.info("this is a params log demo! level is：{}", "info");
	}

{}为参数占位符，“info”为要输出的参数的值。


### 多参数输出日志 ###

	/**
	 * 多参数示例
	 */
	@Test
	public void testMultiParamsLog() {
		logger.info("this is a {} log demo! level is：{}", "multi params", "info");
	}

### 异常输出示例 ###

	/**
	 * 异常信息示例
	 */
	@Test
	public void testExceptionLog() {
		try {
			String errNum = "123456a";
			Integer.parseInt(errNum);
		} catch (Exception e) {
			logger.error("this is a {} demo! ", "exception log", e);
		}
		
	}

###定制输出日志信息###

	/**
	 * MDC定制日志信息
	 */
	@Test
	public void testMDCLog() {
		MDC.put(LogConstants.CURRENT_USERNAME, "admin");
		MDC.put(LogConstants.THREAD_CALLID, Thread.currentThread().getName());
		
		logger.info("this is a mdc log demo!");
	}
注意：MDC中放置的信息的可以在日志输出格式化中声明，格式如下：
[%X{current\_user_name}]


