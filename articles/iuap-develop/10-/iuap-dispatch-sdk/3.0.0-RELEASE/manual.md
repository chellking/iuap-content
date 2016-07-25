# 后台任务客户端SDK组件概述 #

## 业务需求 ##
应用程序中经常会用到跑定时任务的需求，比如定时垃圾回收。有关任务调度需求有时候很复杂，如每隔多长时间重复执行，一个任务在不同时间段执行等。

## 解决方案 ##

任务调度是基于Quartz开发的，具有简单却强大的机制，能依据事件间隔来调度任务，也可以按照cron表达式来操作。任务调度引擎和组件客户端SDK分离，可减小应用服务器的负荷。

iuap-dispatch-sdk是任务调度客户端SDK组件，该组件可通过客户端接口调用服务端的添加、删除、暂停、启动定时任务等功能。

## 功能说明 ##

1.	提供独立的任务调度服务；
2.	支持定时任务执行；
3.	支持重复任务执行；
4.	提供任务的新增、删除、暂停接口。
5.	支持集群环境下任务执行唯一；

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖Quartz框架,其对外提供的依赖方式如下：

```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-dispatch-sdk</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>
```

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能结构 ##
![](images/sdk.jpg)

## 流程说明 ##
- 1、用户通过DispatchRemoteManager接口调度任务。
- 2、DispatchRemoteManager接口向服务端发送调度任务请求。
- 3、服务端实现任务调度。
- 4、客户端通过DispatchClientServlet回调任务。

# 使用说明 #

## 组件包说明 ##

iuap-dispatch-sdk是任务调度客户端SDK组件，该组件可通过客户端接口调用服务端的添加、删除、暂停、启动定时任务等功能。

##组件配置##

**1:配置监听Servlet**

  任务调度客户端的配置文件web.xml中配置监听任务调度客户端SDK提供的servlet类地址，用于执行回调任务。
 
```
    <servlet>
          <servlet-name>DispatchClientServlet</servlet-name>
       	<servlet-class>com.yonyou.iuap.dispatch.client.controller.DispatchClientServlet</servlet-class>
    </servlet>

	<servlet-mapping>
		<servlet-name>DispatchClientServlet</servlet-name>
		<url-pattern>/dispatchClientServlet</url-pattern>
	</servlet-mapping>
```

**2:服务配置**

dispatch-client.properties配置文件字段说明

	dispatch.send.type：任务发送方式，默认HTTP,同时支持SOCKET
	dispatch.server.http.url：服务器地址
	dispatch.recall.type ：任务默认回调方式，默认HTTP,同时支持SOCKET
	dispatch.recall.http.url：任务回调地址

## 工程样例 ##

工程样例可从maven库上下载。例子说明如下：

*(1) 新建任务处理类，需要实现ITask接口*

```
    import java.util.Map;

    import com.yonyou.iuap.dispatch.client.ITask;

    public class TestTaskImpl implements ITask{
	
	
	public void execute(Map<String,String> data) {
		//业务操作
		
	 }
    }
```

*(2) 新增任务*

	**A、简单类型任务**

```
     public void addSimpleTask(Date startDate, TimeConfig timeConfig, String note){
		SimpleTaskConfig taskConfig = 
					new SimpleTaskConfig(UUID.randomUUID().toString()/*任务名*/, 
										 "simpleTaskGroup"/*任务组名*/, 
										 timeConfig);
		taskConfig.setStartDate(startDate);
		Map<String,String> params = new HashMap<String,String>();
		params.put("note", note);
		boolean success = DispatchRemoteManager.add(taskConfig, TestTaskImpl.class, params, true);
		if(success){
			System.out.println("任务添加成功--"+taskConfig.getJobCode());
		}else{
			System.out.println("任务添加失败--"+taskConfig.getJobCode());
		}
	}
 ```
 
    **B、Cron类型任务**
    
```
    public void addCronTask(){
		CronTaskConfig cronTaskConfig = new CronTaskConfig("cronTask", 
														   "cronTaskGroup", 
														   "* */1 * * * ?");
		Map<String,String> params = new HashMap<String,String>();
		params.put("serverName", System.getProperty("os.name"));//任务可传入附加数据
		boolean success = DispatchRemoteManager.add(cronTaskConfig, 
													TestTaskImpl.class, 
													params, 
													true);
		if(success){
			System.out.println("任务添加成功");
		}else{
			System.out.println("任务添加失败");
		}
	}
 
```

## 开发步骤 ##

- 加入依赖
```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-dispatch-sdk</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>
```

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

- 组件配置
  任务调度客户端的配置文件web.xml中需要配置任务调度客户端SDK提供的servlet类地址，用于执行回调任务。
 
 ```
 <servlet>
          <servlet-name>DispatchClientServlet</servlet-name>
       	<servlet-class>com.yonyou.iuap.dispatch.client.controller.DispatchClientServlet</servlet-class>
    </servlet>

	<servlet-mapping>
		<servlet-name>DispatchClientServlet</servlet-name>
		<url-pattern>/dispatchClientServlet</url-pattern>
	</servlet-mapping>
```

dispatch-client.properties配置文件配置

	dispatch.send.type：任务发送方式，默认HTTP,同时支持SOCKET
	dispatch.server.http.url：服务器地址
	dispatch.recall.type ：任务默认回调方式，默认HTTP,同时支持SOCKET
	dispatch.recall.http.url：任务回调地址，客户端所配置的DispatchClientServlet地址



- 编写任务代码
  
该类需要实现ITask接口

- 添加任务

使用SDK提供的DispatchRemoteManager类的add方法添加任务。

## 常用接口 ##

- DispatchRemoteManager

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
			<td>add(SimpleTaskConfig simpleTaskConfig,Class<? extends ITask> taskName, boolean replace)</td>
			<td>simpleTaskConfig简单类型相关参数，taskName任务执行类，replace是否覆盖</td>
			<td>boolean 添加任务是否成功</td>
			<td>添加一个简单类型的任务</td>
		</tr>
		<tr>
			<td>add(CronTaskConfig cronTaskConfig, RecallConfig recallConfig, boolean replace)</td>
			<td>cronTaskConfig 	Cron表达式及相关参数 ，recallConfig回调相关参数，replace是否覆盖</td>
			<td>boolean 添加任务是否成功</td>
			<td>添加一个Cron表达式复杂任务</td>
		</tr>
		<tr>
			<td>deleteJob(String jobName, String groupName)</td>
			<td>jobName 任务名称，groupName组名称</td>
			<td>String 删除成功返回NULL，否则返回具体异常原因</td>
			<td>删除任务</td>
		</tr>
		<tr>
			<td>pauseJob(String jobName, String groupName)</td>
			<td>jobName 任务名称，groupName组名称</td>
			<td>String 暂停成功返回NULL，否则返回具体异常原因</td>
			<td>暂停任务</td>
		</tr>
		<tr>
			<td>resumeJob(String jobName, String groupName)</td>
			<td>jobName 任务名称，groupName组名称</td>
			<td>String 暂停成功返回NULL，否则返回具体异常原因</td>
			<td>启动任务（将任务从暂停状态恢复为执行状态）</td>
		</tr>


	</tbody>
</table>
