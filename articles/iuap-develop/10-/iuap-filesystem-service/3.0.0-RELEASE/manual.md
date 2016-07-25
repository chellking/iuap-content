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

4.pub_filesystem.sql：建表sql语句，需要在数据库里执行的sql。

```
	 CREATE TABLE `pub_filesystem` (
	  `id` varchar(36) NOT NULL ,
	  `pkfile` varchar(100) NOT NULL,
	  `filename` varchar(100) NOT NULL,
	  `filepath` varchar(100) NOT NULL,
	  `filesize` varchar(100) NOT NULL,
	  `groupname` varchar(100) NOT NULL,
	   `permission` varchar(20) NOT NULL,
	  `uploader` varchar(36),
	  `uploadtime` varchar(100),
	  `sysid` varchar(100),
	  `tenant` varchar(100),
	  `modular` varchar(100),
	   `url` varchar(1000),
	  PRIMARY KEY (`id`)
	  ) ENGINE=InnoDB AUTO_INCREMENT=35 DEFAULT CHARSET=utf8;

```

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

### 调用附件方法示例

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

## API接口 ##

**接口返回值参数说明**
~~~
  data：{"message":"失败详情","status":0,"data":[]}
  status：状态，1－成功，2-失败，－1-警告
  data：返回的数据，json格式，具体内容详见下面接口说明
  message：成功或错误信息，一般为成功提示或错误描述，当status为－1时，返回具体批量操作的详细错误，格式如下［{"ObjectName":"校验的对象名称","Field":“校验的字段名称”,"RejectedValue":“错误原因”}］
~~~

1. **上传附件：**filesystem_upload(parameter,callback) 
   <table> 
   <tr>
      	<th></th>
      	<th>参数</th>
      	<th>参数说明</th>
  	</tr>
    <tr>
      	<th>parameter</th>
      	<th>fileElementId</th>
      	<th>【必填】一般为为要提交文件的input标签的ID</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>filepath</th>
      	<th>【必填】附件路径，单据相关的唯一标示，一般为单据ID</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>groupname</th>
     	<th>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>permission</th>
     		<th>read/private，read－公有（即文件不受权限控制，有路径即可访问），private－私有（访问文件前需要调用附件的getURL方法获取认证信息，否则无法访问），不传的时候会默认private</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>purl</th>
     		<th>true/false，是否返回附件的url地址</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>thumbnail</th>
     		<th>缩略图大小设置，例如“500w”，一般仅图片文件使用和url参数配合使用</th>	</tr>
  	<tr>
  		<th></th>
  		<th>isreplace</th>
     		<th></th>
  	</tr>
  	 <tr>
  		<th>return</th>
  		<th>data</th>
     	<th>附件信息数组：格式如下［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}］</th>
  	</tr>
  </table>

2.  **查询附件：**filesystem_query(parameter,callback)
   <table>
  	<tr>
      	<th></th>
      	<th>参数</th>
      	<th>参数说明</th>
  	</tr>
   	<tr>
      	<th>parameter</th>
      	<th>filepath</th>
      	<th>【必填】单据相关的唯一标示，一般为单据ID</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>groupname</th>
     	<th>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度</th>
  	</tr>
  	<tr>
  		<th>return</th>
  		<th>data</th>
     	<th>附件信息数组：格式如下［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}］</th>
  	</tr>
  </table>

3.  **下载附件：**filesystem_download(parameter,callback) 
   <table>
  	<tr>
      	<th></th>
      	<th>参数</th>
      	<th>参数说明</th>
  	</tr>
   	<tr>
      	<th>parameter</th>
      	<th>id</th>
      	<th>【必填】附件的ID</th>
  	</tr>
  </table>
  **注:一般下载附件推荐使用getUrl方法，获取下载地址后直接访问**
4. **删除附件：**filesystem_delete(parameter,callback) 
   <table>
  	<tr>
      	<th></th>
      	<th>参数</th>
      	<th>参数说明</th>
  	</tr>
   	<tr>
      	<th>parameter</th>
      	<th>id</th>
      	<th>【必填】附件的ID</th>
  	</tr>
  </table>
  **注：如果被附件被引用，只删除本条数据，不删除附件**
5. **获得附件的url：**filesystem_geturl(parameter,callback) 
   <table>
  	<tr>
      	<th></th>
      	<th>参数</th>
      	<th>参数说明</th>
  	</tr>
   	<tr>
      	<th>parameter</th>
      	<th>ids</th>
      	<th>【必填】附件的ID数组，格式如下"id1,id2,id3..."</th>
  	</tr>
   	<tr>
      	<th></th>
      	<th>thumbnail</th>
      	<th>缩略图大小，图片时使用，格式如下“100w”</th>
  	</tr>
  </table>
6. **覆盖上传：**filesystem_replace(parameter,callback) 
   <table>
  	<tr>
      	<th></th>
      	<th>参数</th>
      	<th>参数说明</th>
  	</tr>
    <tr>
      	<th>parameter</th>
      	<th>fileElementId</th>
      	<th>【必填】一般为为要提交文件的input标签的ID</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>filepath</th>
      	<th>【必填】附件路径，单据相关的唯一标示，一般为单据ID</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>groupname</th>
     	<th>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>permission</th>
     		<th>read/private，read－公有（即文件不受权限控制，有路径即可访问），private－私有（访问文件前需要调用附件的getURL方法获取认证信息，否则无法访问），不传的时候会默认private</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>purl</th>
     		<th>true/false，是否返回附件的url地址</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>thumbnail</th>
     		<th>缩略图大小设置，例如“500w”，一般仅图片文件使用和url参数配合使用</th>	</tr>
  	<tr>
  		<th></th>
  		<th>isreplace</th>
     		<th></th>
  	</tr>
  	 <tr>
  		<th>return</th>
  		<th>data</th>
     	<th>附件信息数组：格式如下［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}］</th>
  	</tr>
  </table>
  **注：于上传接口基本一致，但会替换原有路径下的文件**
  
7. **附件更新：**filesystem_update(parameter,callback) 
   <table>
  	<tr>
      	<th></th>
      	<th>参数</th>
      	<th>参数说明</th>
  	</tr>
   	<tr>
      	<th>parameter</th>
      	<th>id</th>
      	<th>【必填】附件的ID</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>filepath</th>
      	<th>【必填】附件路径，单据相关的唯一标示，一般为单据ID</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>groupname</th>
     	<th>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度</th>
  	</tr>
  	 <tr>
  		<th>return</th>
  		<th>data</th>
     	<th>附件信息：格式如下{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}</th>
  	</tr>
  </table>
  
8. **直接上传文件服务器：**ossupload(parameter,callback) 
   <table>
  	<tr>
      	<th></th>
      	<th>参数</th>
      	<th>参数说明</th>
  	</tr>
    <tr>
      	<th>parameter</th>
      	<th>fileid</th>
      	<th>【必填】一般为为要提交文件的input标签的ID</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>filepath</th>
      	<th>【必填】附件路径，单据相关的唯一标示，一般为单据ID</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>groupname</th>
     	<th>分組名称,一般在一个单据下存在比较多附件时管理使用，与filepath一起做为文件的管理纬度</th>
  	</tr>
  	<tr>
  		<th></th>
  		<th>permission</th>
     		<th>read/private，read－公有（即文件不受权限控制，有路径即可访问），private－私有（访问文件前需要调用附件的getURL方法获取认证信息，否则无法访问），不传的时候会默认private</th>
  	</tr>
  	<tr>
  		<th>return</th>
  		<th>data</th>
     	<th>附件信息数组：格式如下［{"id":"附件id","filename":“文件名”,"filesize":“附件大小”，“filepath”:“附件的路径”，“groupname”:“分組”，“uploadtime”:“上传时间”，“url”:“下载地址”}］</th>
  	</tr>
  </table>
  **注：目前此方法仅支持使用OSS作为文件服务的方式，使用其他文件服务器时，请不要使用**
