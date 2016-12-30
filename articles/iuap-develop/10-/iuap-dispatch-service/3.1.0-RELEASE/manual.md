# 后台任务服务端组件概述 #

## 业务需求 ##

应用程序中经常会用到跑定时任务的需求，比如定时垃圾回收。有关任务调度需求有时候很复杂，如每隔多长时间重复执行，一个任务在不同时间段执行等。

##解决方案##

本组件提供对Quartz的封装，提供独立的任务调度服务。避免业务系统直接配置Quartz的带来的性能消耗和集群环境的并发问题。 业务系统通过Rest服务添加任务，任务调动执行时回调业务系统的URL启动任务。

iuap-dispatch-service组件功能包括添加、删除、暂停、重启任务。不仅提供了外部调用的Rest服务，并且组件本身也有完整的任务配置界面，包括任务调度，日志查询等功能。

## 功能说明 ##
1.	提供独立的任务调度服务；
2.	支持定时任务执行；
3.	支持重复任务执行；
4.	提供Rest服务对任务进行操作，包括，添加、删除、暂停、重启任务等；
5.	支持集群环境下任务执行唯一；
6.	提供任务配置与管理API和界面；
7.	任务支持分组管理；
8.	支持任务日志查询和搜索；


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖Quartz框架,其对外提供的依赖方式如下：
```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-dispatch-service</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>
```

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能结构 ##

![](./images/service.jpg)

## 功能说明 ##

iuap-dispatch-service组件功能包括添加、删除、暂停、重启任务。不仅提供了外部调用的Rest服务，并且组件本身也有完整的任务配置界面，包括任务调度，日志查询等功能。

# 使用说明 #

## 组件包说明 ##

iuap-dispatch-service组件功能包括添加、删除、暂停、重启任务。不仅提供了外部调用的Rest服务，并且组件本身也有完整的任务配置界面，包括任务调度，日志查询等功能。

##组件配置##

**1:在属性文件中，配置数据库连接等信息**
dispatch_dbinfo.properties如下：
```
  jdbc.driverClassName=com.mysql.jdbc.Driver
	jdbc.url=jdbc:my`://IP:PORT/DATABASE?useUnicode=true&characterEncoding=utf-8
	jdbc.username=用户名
	jdbc.password=密码
```

** 注意，war包中不提供数据库驱动，需要手工将驱动放到war的WEB-INF/lib目录下 **

**2:执行数据库脚本，预置数据库表信息**


依次执行examples项目下sql目录中的dll.sql、index.sql、dml.sql建立数据库并初始化数据。

预置数据库表dispatch_taskway的信息，这张表是用户要执行任务的清单，需要用户预置进去，其中url是指你要执行的定时任务，通过HTTP的方式访问。


## 工程样例 ##

组件提供示例工程可从maven库上下载。

## 组件使用说明 ##

任务调度提供两种方式如下：

### 通过Rest任务调度 ###

* 新建任务Rest接口*

 ** A、新增一个简单任务**

*(1)Rest服务URL*
   "http://localhost:8080/iuap-dispatch-service/dispatchserver/add.do"
*(2)参数,格式如下*

```
{"replace":true,"recallConfig":{"data":{},"option":{"url":"http://localhost:8080/iuap-dispatch-service/dispatchserver/pause.do"},"recallType":"HTTP"},"taskConfig":{"triggerType":"SimpleTrigger","jobCode":"22b511e8-1b80-4f4d-b65e-48f52d8aa682","groupCode":"simpleTaskGroup","startDate":1463813876403,"endDate":null,"priority":0,"timeConfig":{"interval":2,"intervalType":"SECOND","isForever":false,"repeatCount":1}}};
```


 ** B、新增一个Cron表达式任务**
 *(1)Rest服务URL*
   *(1)Rest服务URL*
   "http://localhost:8080/iuap-dispatch-service/dispatchserver/add.do"
* (2)参数,格式如下*

```
 "{\"replace\":true, \"recallConfig\":{\"data\":{\"serverName\":\"Windows 2003\"},\"option\":{\"url\":\"http://localhost:8080/iuap-dispatch-service/dispatchserver/pause.do\"},\"recallType\":\"HTTP\"}, \"taskConfig\":{\"cronExpress\":\"* */1 * * * ?\",\"groupCode\":\"cronTaskGroup\",\"jobCode\":\"cronTask\",\"priority\":0,\"triggerType\":\"CronTrigger\"}}";
```

### 通过界面进行任务调度 ###
将组件war包部署到服务器上访问首页http://IP:PORT/iuap-dispatch-service进行任务的添加.

#### 任务列表界面

界面如下图所示：

![](./images/tasklist.png)

操作说明：可以新增分组、新增任务、查看日志、对任务进行批量的启用停用。

界面说明如下：
- 左侧为任务分组
- 右侧为分组下的任务列表界面，任务会根据状态显示为不同的颜色。最查看任务、查看任务日志、删除任务等操作。鼠标悬停时，未超期任务显示停用、启用操作按钮，

#### 新增任务分组

界面如下图所示：

![](./images/addgroup.png)

操作说明：在主界面上点击界面左上角新建分组按钮。弹出界面，填写后保存。

界面说明如下：
- 分组名称：分组的标识

#### 新增任务

界面如下图所示：

![](./images/addtask.png)

操作说明：在主界面上点击界面左上角新建任务按钮。弹出任务编辑界面，填写后保存。

界面说明如下：

- 任务编码：任务的标识
- 任务名称：具有识别意思的名字
- 任务分组：指定任务所属的组
- 消息接收：配置任务成功后发送消息的接收人和接收方式（需要扩展实现消息发送逻辑）。
- 定时规则：任务具体执行时间，每多少天执行一次，并指定开始时间
- 调用规则：指定要执行的任务，具体规则详见下面预制数据说明。
- 参数：任务执行时，会根据指定的参数值调用业务服务。

定时规则界面如下图所示：

![](./images/timerule.png)

界面说明如下：
- 支持指定任务开始结束时间
- 支持指定每天、每周（可选择周几）、每月第几天执行任务
- 支持在上面选择的日期一天内，指定发生次数、间隔时间和起止时间

#### 查看任务详情

界面如下图所示：

![](./images/taskdetail.png)

操作说明：点击主界面列表的数据行右侧查看任务详情图标。弹出任务详情界面。

界面说明如下：
- 左侧显示任务的名称和执行情况
- 右侧显示任务的详细信息
- 编辑：进入任务编辑界面
- 日志：进入日志查看界面，显示当前任务的日志


#### 修改任务

界面如下图所示：

![](./images/edittask.png)

操作说明：在主界面列表的数据行右侧点击编辑任务图标。弹出任务编辑界面，填写后保存。

界面说明如下：

- 任务编码：任务的标识
- 任务名称：具有识别意思的名字
- 任务分组：指定任务所属的组
- 消息接收：配置任务成功后发送消息的接收人和接收方式（需要扩展实现消息发送逻辑）。
- 定时规则：任务具体执行时间，每多少天执行一次，并指定开始时间
- 调用规则：指定要执行的任务，具体规则详见下面预制数据说明。
- 参数：任务执行时，会根据指定的参数值调用业务服务。

#### 删除任务


操作说明：在主界面列表的数据行右侧点击删除任务图标。

#### 启用任务

界面如下图所示：

![](./images/enabletask.png)

操作说明：鼠标选择已停用的任务时，列表行显示启用按钮。点击按钮启用该任务。

#### 停用任务

界面如下图所示：

![](./images/disabletask.png)

操作说明：鼠标选择正在运行中的任务时，列表行显示停用按钮。点击按钮停用该任务。

#### 批量操作任务

界面如下图所示：

![](./images/multiop.png)

操作说明：点击多选按钮时，会滑出启用、停用、删除按钮。列表界面切换为多选状态，选择多个任务后，点击按钮进行操作。点击多选回到原有状态。

#### 查看任务执行日志

界面如下图所示：

![](./images/viewlog.png)

操作说明：
- 点击主界面左上角的“日志”按钮，显示全部任务日志
- 点击任务详情界面的“日志”按钮，显示当前任务的日志
- 在日志界面的左上角可以选择要查询的任务，选择后，根据指定的任务查询并显示日志信息详见下图。

过滤任务界面：
![](./images/logfilter.png)

操作说明：
- 选择任务分组显示分组下的任务
- 选择具体任务后，界面会按照选择的任务过滤显示。

#### 查看日志详情

界面如下图所示：

![](./images/logdetail.png)

操作说明：点击任务日志列表界面的数据行右侧的“查看原因”按钮，显示日志详情。

- 描述信息：对任务的简单描述


## 开发步骤 ##
- 从maven库上下载war包。
- 配置dispatch_dbinfo.properties 配置文件，连接mysql数据库的数据源信息。
- 执行dispatch.sql 和tables_mysql.sql 初始化数据库的脚本。
- 预置数据库表dispatch_taskway的信息，这张表是用户要执行任务的清单，需要用户预置进去，其中url是指你要执行的定时任务，通过HTTP的方式访问。如果不需要带界面的任务调度服务可以忽略此项。
- Rest服务调用接口提供任务的增删等功能，具体调用方式参考工程样例章节。
- 带界面的任务调度系统访问方式
http://IP:PORT/iuap-dispatch-service


### 预制数据说明 ###
需要用户在数据库表dispatch_taskwayclass、dispatch_taskway和dispatch_taskparam中预制规则数据，
- dispatch_taskwayclas：规则分类
- dispatch_taskway：规则，主要是url字段为执行任务时，调用的业务服务的地址，需要按任务调用的要求提供。
- dispatch_taskparam：规则可设置参数列表，在调用业务服务时，会根据任务指定参数调用

## API接口 ##

### 指定任务Beanid新增基于Cron表达式的定时任务 ###

**描述**

添加或覆盖基于Cron表达式的定时调度任务

**请求方法**

/dispatchserver/add.do

**请求方式**

post

**请求参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
		<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>recallConfig</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>回调信息配置，Json格式，具体参考下面说明</td>
  </tr>
  <tr>
    <td>taskConfig</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>任务执行配置，Json格式，具体参考下面说明</td>
  </tr>
  <tr>
    <td>replace</td>
    <td>True</td>
		<td>boolean</td>
    <td>无</td>
    <td>是否覆盖，为ture时，自动覆盖，为false时，如果存在会返回错误信息</td>
  </tr>
</table>


recallConfig内容格式：

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
		<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>recallType</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>回调方式，SOCKET/HTTP</td>
  </tr>
  <tr>
    <td>option</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>回调的方式地址，"recallType" = "HTTP"时，格式为{"url":"http://ip:port/XXX"}
		<br>"recallType" = "SOCKET"时，格式为{"host":"ip","port"}
		</td>
  </tr>
  <tr>
    <td>data</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>任务附加数据，执行时传递给调用接口，建议为json格式</td>
  </tr>
</table>

taskConfig内容格式：

<table>
  <tr>
    <th>参数字段</th>
    <th>triggerType</th>
    <th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>triggerType</td>
    <td></td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>触发方式，SimpleTrigger/CronTrigger，下面参数根据类型不同变化，
		<br>可参考sdk中的CronTaskConfig/SimpleTaskConfig两种配置类参数
		</td>
  </tr>
  <tr>
    <td>jobCode</td>
    <td>SimpleTrigger/CronTrigger</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>任务名称</td>
  </tr>
  <tr>
    <td>groupCode</td>
    <td>SimpleTrigger/CronTrigger</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>组名称</td>
  </tr>
  <tr>
    <td>priority</td>
    <td>SimpleTrigger/CronTrigger</td>
    <td>True</td>
		<td>int</td>
    <td>无</td>
    <td>优先级，数字大的优先执行</td>
  </tr>
  <tr>
    <td>endDate</td>
    <td>SimpleTrigger/CronTrigger</td>
    <td>True</td>
		<td>Date</td>
    <td>无</td>
    <td>任务结束始时间</td>
  </tr>
  <tr>
    <td>cronExpress</td>
    <td>CronTrigger</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>表达式，</td>
  </tr>
  <tr>
    <td>startDate</td>
    <td>SimpleTrigger</td>
    <td>True</td>
		<td>Date</td>
    <td>无</td>
    <td>任务开始时间</td>
  </tr>
  <tr>
    <td>timeConfig</td>
    <td>SimpleTrigger</td>
    <td>True</td>
		<td>Json</td>
    <td>无</td>
    <td>时间配置，json格式，参考TimeConfig类格式如下</td>
  </tr>
</table>

timeConfig内容格式：

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
		<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>interval</td>
    <td>True</td>
		<td>int</td>
    <td>无</td>
    <td>执行间隔时间，具体单位见intervalType</td>
  </tr>
  <tr>
    <td>intervalType</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>执行间隔时间单位：NULL/MILLISECOND/SECOND/MINUTE/HOUR</td>
  </tr>
  <tr>
    <td>isForever</td>
    <td>True</td>
		<td>boolean</td>
    <td>无</td>
    <td>是否重复执行，为true时会一直定时执行，直到暂停或删除任务</td>
  </tr>
  <tr>
    <td>repeatCount</td>
    <td>True</td>
		<td>int</td>
    <td>无</td>
    <td>重复次数，isForever为false时，任务仅执行指定的次数</td>
  </tr>
</table>


**返回参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>success</td>
    <td>boolean</td>
    <td>true/false</td>
  </tr>
  <tr>
    <td>error</td>
    <td>String</td>
    <td>错误信息</td>
  </tr>
  <tr>
    <td>resultValue</td>
    <td>String</td>
    <td>返回值，一般为null</td>
  </tr>
</table>

### 暂停任务 ###

**描述**

 根据任务名称和组名称暂停任务

**请求方法**

/dispatchserver/pause.do

**请求方式**

post

**请求参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
		<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>jobName</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>任务名称</td>
  </tr>
  <tr>
    <td>groupName</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>组名称</td>
  </tr>
</table>

**返回参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>success</td>
    <td>boolean</td>
    <td>true/false</td>
  </tr>
  <tr>
    <td>error</td>
    <td>String</td>
    <td>错误信息</td>
  </tr>
  <tr>
    <td>resultValue</td>
    <td>String</td>
    <td>返回值，一般为null</td>
  </tr>
</table>

### 恢复任务 ###

**描述**

 根据任务名称和组名称恢复任务

**请求方法**

/dispatchserver/resumeTask.do

**请求方式**

post

**请求参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
		<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>jobName</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>任务名称</td>
  </tr>
  <tr>
    <td>groupName</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>组名称</td>
  </tr>
</table>

**返回参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>success</td>
    <td>boolean</td>
    <td>true/false</td>
  </tr>
  <tr>
    <td>error</td>
    <td>String</td>
    <td>错误信息</td>
  </tr>
  <tr>
    <td>resultValue</td>
    <td>String</td>
    <td>返回值，一般为null</td>
  </tr>
</table>

### 删除任务 ###

**描述**

 根据任务名称和组名称删除任务

**请求方法**

/dispatchserver/delete.do

**请求方式**

post

**请求参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
		<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>jobName</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>任务名称</td>
  </tr>
  <tr>
    <td>groupName</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>组名称</td>
  </tr>
</table>

**返回参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>success</td>
    <td>boolean</td>
    <td>true/false</td>
  </tr>
  <tr>
    <td>error</td>
    <td>String</td>
    <td>错误信息</td>
  </tr>
  <tr>
    <td>resultValue</td>
    <td>String</td>
    <td>返回值，一般为null</td>
  </tr>
</table>

### 立即执行任务 ###

**描述**

 根据任务名称和组名称立即执行任务

**请求方法**

/dispatchserver/trigger.do

**请求方式**

post

**请求参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>必选</th>
		<th>类型</th>
    <th>长度限制</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>jobName</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>任务名称</td>
  </tr>
  <tr>
    <td>groupName</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>组名称</td>
  </tr>
</table>

**返回参数说明**

<table>
  <tr>
    <th>参数字段</th>
    <th>类型</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>success</td>
    <td>boolean</td>
    <td>true/false</td>
  </tr>
  <tr>
    <td>error</td>
    <td>String</td>
    <td>错误信息</td>
  </tr>
  <tr>
    <td>resultValue</td>
    <td>String</td>
    <td>返回值，一般为null</td>
  </tr>
</table>
