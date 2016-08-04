#附件系统服务组件概述

## 业务需求 ##

提供业务系统与文件服务之间的管理功能，支持单据的上传多个附件的管理功能。


## 功能说明 ##

1.	独立的附件服务系统，支持基本的增、删、改、等等，可以和其他组件集成。
2.	支持多种格式的附件上传
3.	支持一次选择多个附件
4.	支持直传回调、支持回调自定义参数的扩展、支持oss和FastDFS、支持缩略图调节；
5.	支持微服务的部署方式；

#整体设计#

##依赖环境 ##

```
<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-filesystem-service</artifactId>
	  <version>${iuap.modules.version}</version>
	  <type>war</type>
	</dependency>
```
${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能结构 ##

本组件依赖于文件组件，通过文件组件可适配不同的文件服务器，目前支持FastDFS，阿里的OSS服务。
本组件主要用于业务系统的附件管理功能，提供具体业务单据于单据下多个附件的分组管理功能。通过附件管理表建立业务单据与具体文件的关系，支持业务单据对文件的管理功能。同时屏蔽了不同文件服务器不同接口调用。

# 使用说明 #

##配置说明：

1.application.properties：阿里云、FastDFS 配置文件，具体可参考文件组件使用说明

```
  #使用的存储类型，选项为FastDfs或AliOss
  storeType=FastDfs
	#----------------------------------------------------------------------------------------------
	#使用FastDfs文件系统时Fdfs系统的配置******
	connect_timeout =
	network_timeout =
	charset =
	tracker_server =
	fdfsread_server =
	#----------------------------------------------------------------------------------------------
	#aliyun 采用阿里的文件系统配置
	storeDir=
	endpoint=
	#accessKeyId 是阿里附件系统提供的账户信息
	accessKeyId=
	accessKeySecret=
	#回调的服务器地址，使用直传的时候必须配置
	callbackTarget=http://ip:端口
  #回调方法:一般不用修改，开发人员扩展使用
	callbackUrl=/file/rewrite
	callbackBody=filename=${object}&bucket=${bucket}&size=${size}&groupname=${x:groupname}&filepath=${x:filepath}&permission=${x:permission}&modular=${x:modular}

```

2.iuapfile.properties：阿里云bucket信息配置文件，仅在使用AliOss作为文件服务时使用

```
    #私有bucket
    defaultBucket=阿里文件存储的私有账户
    #可读bucket==公有
    defaultBucketRead=阿里文件存储的公有账户（read权限账户）
```

3.jdbc.properties：数据库配置文件

```
    #驱动
    jdbc.driver=com.mysql.jdbc.Driver
    #url地址
	jdbc.url=jdbc:mysql://localhost:3306/ssm
    #数据库用户
	jdbc.username=mysql
	#数据库密码
	jdbc.password=mysql
	#定义初始连接数
	jdbc.initialSize=1
	#定义最大连接数
	jdbc.maxActive=20
	#定义最大空闲
	jdbc.maxIdle=20
	#定义最小空闲
	jdbc.minIdle=1
	#定义最长等待时间
	jdbc.maxWait=60000
```

** 注意，war包中不提供数据库驱动，需要手工将驱动放到war的WEB-INF/lib目录下 **

4.执行数据库脚本

依次执行examples项目下sql目录中的dll.sql、index.sql、dml.sql建立数据库并初始化数据。


## 示例工程说明：##

**文件说明：**
      index.jsp ---使用示例，包含说明信息。

      ossupload.js ---直传的js文件
      ajaxfileupload.js ---附件上传插件js,不支持跨域。跨域模式在index.jsp中存在例子
      interface.file.js ---ajax访问方式 接口文件，里面是需要实现的方法
      interface.file.impl.js ---是interface.file.js的实现，包括所有的基本js
      【以上js文件使用的时候需要同时引用这4个js,顺序为ossupload.js、ajaxfileupload.js、interface.file.js、interface.file.impl.js】


## 开发步骤 ##


### 导入js文件

```
		<script type="text/javascript" src="<%=request.getContextPath()%>/resources/js/ossupload.js"></script>
		<script type="text/javascript" src="<%=request.getContextPath()%>/resources/js/ajaxfileupload.js"></script>
		<script type="text/javascript" src="<%=request.getContextPath()%>/resources/js/interface.file.js"></script>
		<script type="text/javascript" src="<%=request.getContextPath()%>/resources/js/interface.file.impl.js"></script>
```

### web前端调用附件方法示例

1.   基本方法：以上传附件举例

```
	function upload(){
		 var par = {
				 fileElementId: "uploadbatch_id",  
				 filepath: "code",   
				 groupname: "single",
				 permission: "read",
				 url: true,          
				 thumbnail :  "500w",				 }
		 var f = new interface_file();
		 f.filesystem_upload(par,callback);//callback是上面定义的回调函数	 }
```

2.  回调函数：处理过程，一遍业务编写，

代码示例如下：

```
		/** * 回调函数--返回结果*/
		 var callback = function(data){
			 if(-1 == data.status){//后台校验信息状态
				 var warn = "";
				 for(var k in  data.message){
					 data.message[k].ObjectName    //校验的对象名称
					 data.message[k].Field         //校验的字段名称
					 data.message[k].RejectedValue //校验的错误原因		 
					 data.message[k].DefaultMessage; //校验的错误提示信息
					 warn +=data.message[k].ObjectName+"对象的"+ data.message[k].Field+"属性不能为："+data.message[k].RejectedValue +"\n"; //自己拼的方式
					 warn +=data.message[k].DefaultMessage +"\n"//后台的默认信息
				 }
				 alert(warn);
			 }else if(1 == data.status){//上传成功状态
         //业务代码
			 }else if(0 == data.status){//上传失败状态
         //业务代码
			 }else{//error 或者加載js錯誤

				  alert(data);
			 }
 	};

```


### 登录用户的信息
默认的用户传递方式时cookies中，指定usercode为用户的编码或主键，如果业务上使用其他的cookies值，请重写fileserver-spring-mvc.xml中的拦截器，

```
		<mvc:interceptors>  
	      <!-- session超时 -->  
	      <mvc:interceptor>  
	        <mvc:mapping path="/*/*"/>  
	        <bean class="com.yonyou.iuap.generic.adapter.CookiesInterceptor">
	          <property name="exclude">  
	            <list>  
	              <!-- 如果请求中包含以下路径，则不进行拦截 -->  
	              <value>/login</value>  
	              <value>/js</value>  
	              <value>/css</value>  
	              <value>/image</value>  
	              <value>/images</value>  
	            </list>  
	          </property>  
	        </bean>  
	      </mvc:interceptor>  
	    </mvc:interceptors>
```

在拦截器中使用InvocationInfoProxyAdapter.setUserid(userid);设置用户编码或主键。

### interface.file.js ###

在web前端的代码中直接使用以下接口方法

#### 附件上传

**描述**  

附件上传  

**请求方法**  

filesystem_upload(parameter,callback)

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
			<th>fileElementId</th>
	    <td>True</td>
			<td>String</td>
	    <td>无</td>
			<th>一般为为要提交文件的input标签的ID</th>
	</tr>
  <tr>
    <td>filepath</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>附件路径，单据相关的唯一标示，一般为单据ID</td>
  </tr>
  <tr>
    <td>groupname</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度<</td>
  </tr>
  <tr>
    <td>permission</td>
    <td>False</td>
		<td>String</td>
    <td>20</td>
    <td>read/private，read－公有（即文件不受权限控制，有路径即可访问），private－私有（访问文件前需要调用附件的getURL方法获取认证信息，否则无法访问），不传的时候会默认private</td>
  </tr>
  <tr>
    <td>url</td>
    <td>False</td>
		<td>boolean</td>
    <td>无</td>
    <td>是否返回附件的url地址</td>
  </tr>
  <tr>
    <td>thumbnail</td>
    <td>False</td>
		<td>String</td>
    <td>10</td>
    <td>缩略图大小设置，例如“500w”，一般仅图片文件使用和url参数配合使用</td>
  </tr>
  <tr>
    <td>isreplace</td>
    <td>False</td>
		<td>boolean</td>
    <td>无</td>
    <td>是否覆盖，当filepath，groupname和filename都相同时是否覆盖掉之前上传的附件，默认为:false,</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>返回的数据，json格式，
				<br>具体内容格式如下：［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}］
		</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>

#### 附件查询

**请求方法**  

filesystem_query(parameter,callback)

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
    <td>filepath</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>附件路径，单据相关的唯一标示，一般为单据ID</td>
  </tr>
    <tr>
    <td>groupname</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度<</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>返回的数据，json格式，
				<br>具体内容格式如下：［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}］
		</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>

#### 附件下载  

**请求方法**  

filesystem_download(parameter,callback)

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
    <td>id</td>
    <td>True</td>
		<td>String</td>
    <td>最长36,一般19</td>
    <td>附件的ID</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>具体文件对象</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>

#### 附件删除  

**请求方法**  

filesystem_delete(parameter,callback)

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
    <td>id</td>
    <td>True</td>
		<td>String</td>
    <td>最长36,一般19</td>
    <td>附件的ID</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>空</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>

#### 获得附件的url

**请求方法**  

filesystem_geturl(parameter,callback)

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
    <td>ids</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>附件的ID数组，格式如下"id1,id2,id3..."</td>
  </tr>
  <tr>
    <td>thumbnail</td>
    <td>False</td>
		<td>String</td>
    <td>10</td>
    <td>缩略图大小设置，例如“500w”，一般仅图片文件使用和url参数配合使用</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>空</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>


#### 附件覆盖上传

**请求方法**  

filesystem_replace(parameter,callback)

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
    <td>fileElementId</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>一般为为要提交文件的input标签的ID</td>
	</tr>
  <tr>
    <td>id</td>
    <td>True</td>
		<td>String</td>
    <td>最长36,一般19</td>
    <td>需要被覆盖的附件ID</td>
  </tr>
  <tr>
    <td>filepath</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>附件路径，单据相关的唯一标示，一般为单据ID</td>
  </tr>
   <tr>
    <td>groupname</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度<</td>
  </tr>
  <tr>
	 <td>permission</td>
	 <td>False</td>
	 <td>String</td>
	 <td>20</td>
	 <td>read/private，read－公有（即文件不受权限控制，有路径即可访问），private－私有（访问文件前需要调用附件的getURL方法获取认证信息，否则无法访问），不传的时候会默认private</td>
  </tr>
  <tr>
		<td>url</td>
		<td>False</td>
		<td>boolean</td>
		<td>无</td>
		<td>是否返回附件的url地址</td>
  </tr>
  <tr>
		<td>thumbnail</td>
		<td>False</td>
		<td>String</td>
		<td>10</td>
		<td>缩略图大小设置，例如“500w”，一般仅图片文件使用和url参数配合使用</td>
   </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>返回的数据，json格式，
				<br>具体内容格式如下：［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}
		</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
			</td>
  </tr>
</table>


#### 附件更新

**请求方法**  

filesystem_update(parameter,callback)

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
    <td>id</td>
    <td>True</td>
		<td>String</td>
    <td>最长36,一般19</td>
    <td>需要被覆盖的附件d的ID</td>
  </tr>
  <tr>
    <td>filepath</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>附件路径，单据相关的唯一标示，一般为单据ID</td>
  </tr>
  <tr>
    <td>groupname</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度<</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>返回的数据，json格式，
				<br>具体内容格式如下：［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}
		</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
			</td>
  </tr>
</table>

#### 直接上传文件服务器

**请求方法**  

ossupload(parameter,callback)

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
    <td>fileid</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>一般为为要提交文件的input标签的ID</td>
	</tr>
  <tr>
    <td>filepath</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>附件路径，单据相关的唯一标示，一般为单据ID</td>
  </tr>
   <tr>
    <td>groupname</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度<</td>
  </tr>
  <tr>
	 <td>permission</td>
	 <td>False</td>
	 <td>String</td>
	 <td>20</td>
	 <td>read/private，read－公有（即文件不受权限控制，有路径即可访问），private－私有（访问文件前需要调用附件的getURL方法获取认证信息，否则无法访问），不传的时候会默认private</td>
  </tr>
  <tr>
		<td>url</td>
		<td>False</td>
		<td>boolean</td>
		<td>无</td>
		<td>是否返回附件的url地址</td>
  </tr>
  <tr>
		<td>thumbnail</td>
		<td>False</td>
		<td>String</td>
		<td>10</td>
		<td>缩略图大小设置，例如“500w”，一般仅图片文件使用和url参数配合使用</td>
   </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>返回的数据，json格式，
				<br>具体内容格式如下：［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}
		</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
			</td>
  </tr>
</table>

### 附件管理RestAPI ###

#### 附件上传

**描述**  
附件上传  
**请求方法**  

file/upload

**请求方式**  

POST

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
    <td>filepath</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>附件路径，单据相关的唯一标示，一般为单据ID</td>
  </tr>
  <tr>
    <td>groupname</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度<</td>
  </tr>
  <tr>
    <td>permission</td>
    <td>False</td>
		<td>String</td>
    <td>20</td>
    <td>read/private，read－公有（即文件不受权限控制，有路径即可访问），private－私有（访问文件前需要调用附件的getURL方法获取认证信息，否则无法访问），不传的时候会默认private</td>
  </tr>
  <tr>
    <td>url</td>
    <td>False</td>
		<td>boolean</td>
    <td>无</td>
    <td>是否返回附件的url地址</td>
  </tr>
  <tr>
    <td>thumbnail</td>
    <td>False</td>
		<td>String</td>
    <td>10</td>
    <td>缩略图大小设置，例如“500w”，一般仅图片文件使用和url参数配合使用</td>
  </tr>
  <tr>
    <td>isreplace</td>
    <td>False</td>
		<td>boolean</td>
    <td>无</td>
    <td>是否覆盖，当filepath，groupname和filename都相同时是否覆盖掉之前上传的附件，默认为:false,</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>返回的数据，json格式，
				<br>具体内容格式如下：［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}］
		</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>

#### 附件查询

**请求方法**  

file//query

**请求方式**  

GET

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
    <td>filepath</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>附件路径，单据相关的唯一标示，一般为单据ID</td>
  </tr>
    <tr>
    <td>groupname</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度<</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>返回的数据，json格式，
				<br>具体内容格式如下：［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}］
		</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>

#### 附件下载  

**请求方法**  

file/download

**请求方式**  

GET

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
    <td>id</td>
    <td>True</td>
		<td>String</td>
    <td>最长36,一般19</td>
    <td>附件的ID</td>
  </tr>
  <tr>
    <td>stream</td>
    <td>false</td>
		<td>String</td>
    <td>10</td>
    <td>"true"</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>具体文件对象</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>

#### 附件删除  

**请求方法**  

file/delete

**请求方式**  

GET

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
    <td>id</td>
    <td>True</td>
		<td>String</td>
    <td>最长36,一般19</td>
    <td>附件的ID</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>空</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>

#### 获得附件的url

**请求方法**  

file/url

**请求方式**  

GET

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
    <td>ids</td>
    <td>True</td>
		<td>String</td>
    <td>无</td>
    <td>附件的ID数组，格式如下"id1,id2,id3..."</td>
  </tr>
  <tr>
    <td>thumbnail</td>
    <td>False</td>
		<td>String</td>
    <td>10</td>
    <td>缩略图大小设置，例如“500w”，一般仅图片文件使用和url参数配合使用</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>空</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
		</td>
  </tr>
</table>

#### 附件覆盖上传

**请求方法**  

file/replace

**请求方式**  

POST

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
    <td>id</td>
    <td>True</td>
		<td>String</td>
    <td>最长36,一般19</td>
    <td>需要被覆盖的附件ID</td>
  </tr>
  <tr>
    <td>filepath</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>附件路径，单据相关的唯一标示，一般为单据ID</td>
  </tr>
   <tr>
    <td>groupname</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度<</td>
  </tr>
  <tr>
	 <td>permission</td>
	 <td>False</td>
	 <td>String</td>
	 <td>20</td>
	 <td>read/private，read－公有（即文件不受权限控制，有路径即可访问），private－私有（访问文件前需要调用附件的getURL方法获取认证信息，否则无法访问），不传的时候会默认private</td>
  </tr>
  <tr>
		<td>url</td>
		<td>False</td>
		<td>boolean</td>
		<td>无</td>
		<td>是否返回附件的url地址</td>
  </tr>
  <tr>
		<td>thumbnail</td>
		<td>False</td>
		<td>String</td>
		<td>10</td>
		<td>缩略图大小设置，例如“500w”，一般仅图片文件使用和url参数配合使用</td>
   </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>返回的数据，json格式，
				<br>具体内容格式如下：［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}
		</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
			</td>
  </tr>
</table>


#### 附件更新

**请求方法**  

file/replace

**请求方式**  

POST

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
      <td>id</td>
    <td>True</td>
	<td>String</td>
    <td>最长36,一般19</td>
    <td>需要被覆盖的附件d的ID</td>
  </tr>
  <tr>
    <td>filepath</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>附件路径，单据相关的唯一标示，一般为单据ID</td>
  </tr>
  <tr>
    <td>groupname</td>
    <td>True</td>
		<td>String</td>
    <td>100</td>
    <td>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度<</td>
  </tr>
</table>

**返回参数说明**  

返回："data":{"message":"提示信息","status":0,"data":[]}

<table>
  <tr>
    <th>参数字段</th>
    <th>返回值说明</th>
  </tr>
  <tr>
    <td>status</td>
    <td>状态，1－成功，2-失败，－1-警告</td>
  </tr>
  <tr>
    <td>data</td>
    <td>返回的数据，json格式，
				<br>具体内容格式如下：［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}
		</td>
  </tr>
  <tr>
    <td>message</td>
    <td>返成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，
				<br>格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
			</td>
  </tr>
</table>
