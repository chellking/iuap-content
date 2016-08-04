#本地事件通知组件概述#

## 组件简介

基于插件扩展的同一系统内部的事件通知机制；

可以注册多个监听，按注册指定的顺序依次执行；


## 功能说明 ##

1.	基于插件扩展的同一系统内部的事件通知机制；
2.	支持多个监听，按注册指定的顺序依次执行；
3.	支持事件和监听的注册；


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：

```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-organization</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>
```
${iuap.modules.version} 为平台在maven私服上发布的组件的version。

##数据库表

目前未提供界面需要通过数据库注册的方式。

### 事件类型pub_eventtype 
<table>
   <tr>
      <td>名称</td>
      <td>代码</td>
      <td>注释</td>
   </tr>
   <tr>
      <td>主键</td>
      <td>id</td>
      <td></td>
   </tr>
   <tr>
      <td>事件源编码</td>
      <td>sourceid</td>
      <td>一般用来描述产生事件的对象或单据。例如用户，租户等</td>
   </tr>
   <tr>
      <td>事件源名称</td>
      <td>sourcename</td>
      <td></td>
   </tr>
   <tr>
      <td>事件源名称多语编码</td>
      <td>sourcenamei18n</td>
      <td></td>
   </tr>
   <tr>
      <td>事件类型编码</td>
      <td>eventtypecode</td>
      <td>一般用于描述事件对象或单据的具体行为。例如：新增、修改、删除等</td>
   </tr>
   <tr>
      <td>事件类型名称</td>
      <td>eventtypename</td>
      <td></td>
   </tr>
   <tr>
      <td>事件类型名称多语编码</td>
      <td>eventtypenamei18n</td>
      <td></td>
   </tr>
   <tr>
      <td>详细说明</td>
      <td>note</td>
      <td>主要对事件的消息内容格式做出说明，便于后续监听插件安格式获取数据。</td>
   </tr>
   <tr>
      <td>最后更新时间</td>
      <td>last_time</td>
      <td></td>
   </tr>
   <tr>
      <td>版本</td>
      <td>version</td>
      <td></td>
   </tr>
</table>
	

### 事件通知业务插件表pub_eventlistener

<table>
   <tr>
      <td>名称</td>
      <td>代码</td>
      <td>注释</td>
   </tr>
   <tr>
      <td>主键</td>
      <td>id</td>
      <td></td>
   </tr>
   <tr>
      <td>事件类型主键</td>
      <td>event_type_id</td>
      <td></td>
   </tr>
   <tr>
      <td>插件名称</td>
      <td>name</td>
      <td></td>
   </tr>
   <tr>
      <td>插件名称多语编码</td>
      <td>namei18n</td>
      <td></td>
   </tr>
   <tr>
      <td>插件全类名</td>
      <td>implclassname</td>
      <td>对应插件的类名，需要实现IBusinessEventListener接口</td>
   </tr>
   <tr>
      <td>插件功能备注</td>
      <td>note</td>
      <td></td>
   </tr>
   <tr>
      <td>插件执行顺序号</td>
      <td>operindex</td>
      <td>同一业务系统内顺序号唯一，同一业务系统内事件按插件顺序执行</td>
   </tr>
   <tr>
      <td>是否启用</td>
      <td>enabled</td>
      <td>Y/N</td>
   </tr>
   <tr>
      <td>最后更新时间</td>
      <td>last_time</td>
      <td></td>
   </tr>
   <tr>
      <td>版本</td>
      <td>version</td>
      <td></td>
   </tr>
</table>


# 使用说明 #

## 集成说明

1、服务启动，Spring初始化配置文件路径中添加classpath:eventLocal-applicationContext.xml,请参考示例工程。

2、执行建表脚本

依次执行examples项目下sql目录中的dll.sql、index.sql、dml.sql建立数据库并初始化数据。
	

## 配置说明

组件需要访问数据库，eventLocal-applicationContext.xml文件定义了一个依赖id为dataSource数据源的bean，请在主配置文件中提供id为dataSource的数据源，或者修改配置文件eventlocal-applicationContext.properties，具体配置参数请参考示例工程。

## 使用示例

假定一个事件用户新增前事件：

#####(1)	新增处理插件
需要实现IBussinessListener，如图所示，新增一个处理插件：
com.yonyou.iuap.event.local.test.LocalEventPluginImpl；

 
#####(2)	注册监听
注册事件类型
<pre>
INSERT INTO pub_eventtype (id, sourceid, sourcename, sourcenamei18n, eventtypecode, eventtypename, eventtypenamei18n, note, last_time, version) VALUES ('1', 'USER', '用户', 'user_code', 'ADD_BEFORE', '新增前', 'add_before_code', null, null, 1);
</pre>
注册事件监听插件
<pre>
INSERT INTO pub_eventlistener (id, event_type_id, name, namei18n, implclassname, note, operindex, enabled, last_time, version) VALUES ('1', '1', '测试', 'test_code', 'com.yonyou.iuap.event.local.test.LocalEventPluginImpl', null, 1, 'Y', null, 1);
</pre>
#####(3)	事件触发
<pre>
BusinessEvent event = new BusinessEvent("USER","ADD_BEFORE",System.currentTimeMillis()/*事件要发送的内容，格式是字符型*/);
		EventDispatcher.fireEvent(event);
</pre>
#####（4）监听类实现IBussinessListener

```
    public class LocalEventPluginImpl implements IBussinessListener{
	/**
	 * 事件响应
	 */
	public void doAction(BusinessEvent businessEvent) {
		System.out.println("doAction, time="+(System.currentTimeMillis()-Long.valueOf(String.valueOf(businessEvent.getUserObject())))+",event="+businessEvent);
	}
	
	public void doBackAction(BusinessEvent businessEvent) {
		System.out.println("doBackAction, event="+businessEvent);
	}
    }
 ```