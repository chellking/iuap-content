# UI后端框架 #


## 业务需求

接收并处理前端datatable发送的数据，拥有独立的上下文处理，能够处理前端后端交互过程中的、参数事件、多语、异常，简化开发过程。


## 解决方案 ##
此组件是iuap前端框架的后端适配组件，和spring MVC配合，接收并处理前端UI datatable发送的数据，拥有独立的上下文处理和序列化框架，方便了开发者对前后端数据的对接。

Datatable控制器是一个特殊的spring MVC 控制器，与普通的Spring MVC控制器的区别有两点：

功能说明

1. 通过Datatable获取数据和向前台输出数据
2. 获取前台发送的参数
3. 向前台输出数据

# 整体设计


## 依赖环境

 组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：

	<dependency>
            <groupId>com.yonyou.iuap</groupId>
            <artifactId>iuap-iweb</artifactId>
            <version>${iuap.modules.version}</version>
            <exclusions>
                <exclusion>
                    <artifactId>spring-webmvc</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>cglib-nodep</artifactId>
                    <groupId>cglib</groupId>
                </exclusion>
            </exclusions>
        </dependency>

${iuap.modules.version} 为平台在maven私服上发布的组件的version 

# 使用说明


## 属性配置

1. 在工程的web.xml中配置filter

		
		<filter>
		    <filter-name>IWebPlatformFilter</filter-name>
		    <filter-class>com.yonyou.iuap.iweb.platform.IWebPlatformFilter</filter-class>
		</filter>
		<filter-mapping>
		    <filter-name>IWebPlatformFilter</filter-name>
		    <servlet-name>springServlet</servlet-name>
		</filter-mapping>

2. 在工程的spring MVC的配置文件中加入mvc:annotation-driven配置

		<mvc:annotation-driven>
			<mvc:message-converters register-defaults="true">
				<!-- 将StringHttpMessageConverter的默认编码设为UTF-8 -->
				<bean class="org.springframework.http.converter.StringHttpMessageConverter">
			    	<constructor-arg value="UTF-8" />
				</bean>
				<!-- 将Jackson2HttpMessageConverter的默认格式化输出设为true -->
				<bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
	                <property name="prettyPrint" value="true"/>
	            </bean>			
	  		</mvc:message-converters>
	  		<mvc:argument-resolvers>
	  			<bean class="com.yonyou.iuap.iweb.datatable.handler.IWebHandlerMethodArgumentResolver"/>
	  		</mvc:argument-resolvers>
		</mvc:annotation-driven>
	
	    <bean id="iwebExceptionHandler" class="com.yonyou.iuap.iweb.exception.WebExceptionHandler"/>
## 使用API接口完成请求

## API接口

### Datatable

描述

DataTable作为前后台数据传输的载体，它有着和前端相对应的模型体信息，如：行（Row）、列（Field）、列描述（FieldDesc）、分页；同时它具有和业务bean信息透明穿透的能力；也维护了自身状态和行为的变化。

Datatable属性

<table style="border-collapse:collapse;border-color:#e1e1e1">
	<thead><tr>
		<td>属性</td>
		<td>描述</td>
		<td>类型</td>
	</tr></thead>
	<tbody>
		<tr><td>id</td><td>datable的编码id（一次请求中唯一）</td><td>String</td></tr>
		<tr><td>rows</td><td>当前数据集中的行记录</td><td>List</td></tr>
		<tr><td>cls</td><td>对应的实体class</td><td>String</td></tr>
		<tr><td>select</td><td>选中行索引</td><td>Integer[]</td></tr>
		<tr><td>focus</td><td>聚焦行索引</td><td>Integer</td></tr>
		<tr><td>pageIndex</td><td>当前页号</td><td>Integer</td></tr>
		<tr><td>pageSize</td><td>每页记录数</td><td>Integer</td></tr>
		<tr><td>totalPages</td><td>总页数</td><td>Integer</td></tr>
		<tr><td>totalRow</td><td>总行数</td><td>Long</td></tr>
		<tr><td>isBread</td><td>是否面包屑（携带描述信息不带数据）</td><td>boolean</td></tr>
		<tr><td>meta</td><td>entity描述信息</td><td>Map&lt;String,FieldDesc&gt;</td></tr>
		<tr><td>changedObj</td><td>变化项集,供前端界面更新</td><td>Map&lt;String, Object&gt;</td></tr>
		<tr><td>params</td><td>业务参数扩展</td><td>Map&lt;String, Object&gt;</td></tr>
    </tbody>
</table>

方法：

<table style="border-collapse:collapse;border-color:#e1e1e1">
	<thead><tr>
		<td>方法</td>
		<td>描述</td>
		<td>参数</td>
		<td>参数描述</td>
		<td>返回值</td>
	</tr></thead>
	<tbody>
		<tr><td>
	<a onclick="toggle('getRow')">getRow</a></td>
         <td>根据行号获取行</td><td>String rowid</td><td>行号</td><td>行</td></tr>

	<tr><td><a onclick="toggle('getSelectedRow')">getSelectedRow</a></td><td>获得选中的行</td><td>——</td><td>——</td><td>选中的行</td></tr>

	<tr><td><a onclick="toggle('getFocusRow')">getFocusRow</a></td><td>获得聚焦行</td><td>——</td><td>——</td><td>聚焦行</td></tr>

	<tr><td><a onclick="toggle('createEmptyRow')">createEmptyRow</a></td><td>创建一个新行，由cls生成列信息</td><td>——</td><td>——</td><td>空行</td></tr>
	<tr><td><a onclick="toggle('addRow')">addRow</a></td><td>添加一行记录</td><td>Row row</td><td>一行</td><td>——</td></tr>

	<tr><td><a onclick="toggle('addRows')">addRows</a></td><td>添加多行记录</td><td>List<Row> rows</td><td>多行</td><td>——</td></tr>

	<tr><td><a onclick="toggle('getAll')">getAll</a></td><td>获得所有业务bean集</td><td>——</td><td>——</td><td>POJO对象列表</td></tr>



	<tr><td><a onclick="toggle('updateMeta')">updateMeta</a></td><td>更新元数据描述信息</td><td>String fieldid,String descName,String descValue</td><td>fieldid：列id；descName：描述名称（如：default）；descValue：描述值</td><td>--</td></tr>

	<tr><td><a onclick="toggle('add')">add</a></td><td>添加业务bean</td><td>&lt;T> t</td><td>业务实体bean</td><td>--</td></tr>

	<tr><td><a onclick="toggle('add')">add</a></td><td>添加多个业务bean</td><td>&lt;T>[] t</td><td>多个业务实体bean</td><td>--</td></tr>

	<tr><td><a onclick="toggle('add')">add</a></td><td>添加多个业务bean,并增加添加前后业务处理逻辑</td><td>T[] objs,IAddRowHandler addrowHandler</td><td>objs：多个业务实体bean；addrowHandler：前后处理服务</td><td>--</td></tr>

	<tr><td><a onclick="toggle('getT')">getT</a></td><td>根据row生成包装业务bean</td><td>Row row</td><td>row：一行记录</td><td>POJO对象</td></tr>

	<tr><td><a onclick="toggle('toBean')">toBean</a></td><td>根据row生成bean</td><td>Row row</td><td>row：一行记录</td><td>POJO对象</td></tr>

	<tr><td><a onclick="toggle('toBean')">toBean</a></td><td>为包装bean绑定数据</td><td>T t</td><td>t：包装bean</td><td>POJO对象</td></tr>
	
	<tr><td><a onclick="toggle('breed')">breed</a></td><td>在事件请求上下文里添加一个新的不带数据的datable</td><td>String nid</td><td>nid：datatble编码</td><td>新的datatble</td></tr>

    </tbody>
</table>


### 获取请求参数

描述

获取前端发送过来的参数


请求方法

	IWebViewContext.getEventParameter(key)

请求方式

方法调用

请求参数说明

| 参数字段      |    必选 | 类型  |说明  |
| :-------- | --------:|--------:| :--: |
| key  | true |  字符串   |参数key |


返回值

参数的值


### 获取Datatable

描述

获取前端发送过来的Datatable


请求方法

	IWebViewContext.getRequest().getDataTable(key)

请求方式

方法调用

请求参数说明

| 参数字段      |    必选 | 类型  |说明  |
| :-------- | --------:|--------:| :--: |
| key  | true |  字符串   |datatable 的ID |


返回值

datatable对象


### 向前端输出数据

描述

向前端输出数据


请求方法

	IWebViewContext.getResponse().getWriter().write(value)

请求方式

方法调用

请求参数说明

| 参数字段      |    必选 | 类型  |说明  |
| :-------- | --------:|--------:| :--: |
| value  | true |  字符串   |输出内容 |


返回值

无





