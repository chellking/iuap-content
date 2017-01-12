# 文件组件 #

## 业务需求 ##

业务场景中经常会涉及到和文件的交互，需要能够对文件资源进行有效的操作和管理。但传统的单机文件系统在互联网应用场景下往往效率不高、稳定性不佳、扩展困难，无法达到高并发、高性能的业务需求，所以需要有一种高效稳定的方式来提供文件资源管理。

##解决方案##

iuap的文件组件提供了对文件资源的通用操作，使用分布式文件系统FastDFS高效存储海量文件，通过集群模式水平扩展容量和数据冗余存储，同时支持阿里云OSS对象存储服务，使用OSS直接上传文件可以节省带宽流量。在对不同的文件系统进行适配时，尽量保持接口参数相同，保证用户使用简便。

##功能说明##

1.	支持文件的上传、下载、删除操作；
2.	同时支持FastDFS和阿里云OSS服务；
3.	支持获取文件下载URL；
4.	支持阿里云OSS的文件直传和回调；
5.	支持阿里云OSS不同bucket权限的文件URL获取；
6.	支持对阿里云OSS请求的安全校验；

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖FastDFS和阿里云OSS的客户端SDK,其对外提供的依赖方式如下：

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-file</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能结构 ##


<img src="/images/file_structure.jpg"/>

组件封装了三种文件系统，并暴露出了常用的文件操作接口，用户通过这些接口，去操作组件所管理的对应文件系统。

# 使用说明 #

## 配置 ##
**文件组件目前适配了三种文件系统，本地文件存储、阿里云、FastDfs**

文件组件支持三套文件系统，在使用不同的系统时接口保持统一，通过FileManager类下的静态方法，提供对文件上传下载删除的服务。

**1:工程中引入对iuap-file组件的依赖**

	<dependency>
		<groupId>com.yonyou.iuap</groupId>
		<artifactId>iuap-file</artifactId>
		<version>${iuap.modules.version}</version>
	</dependency>

${iuap.modules.version} 为在pom.xml中定义的需要引入组件的version。

**2:在application.properties文件中配置工程所使用的文件存储系统**


	#使用哪种文件存储系统（AliOss阿里云，Local本地文件存储，FastDfs）
    storeType=AliOss
    #storeType=Local
    #storeType=FastDfs

	#默认存储Bucket(private权限)
    defaultBucket=your bucket
	#默认存储Bucket(read权限)
    defaultBucketRead=your bucket

    #使用本地文件系统时的存储路径
    storeDir=/etc/filetest
	
	#使用FastDfs文件系统时Fdfs系统的配置
    connect_timeout = 10
	network_timeout = 30
	charset = ISO8859-1
	tracker_server = 20.12.6.91:22122
	#处理FastDfs文件系统的nginx服务器ip（文件url获取以及图片缩放功能）
	fdfsread_server = 172.20.7.24

    
    #使用阿里云文件系统
	#阿里云endpoint、accessKeyId、accessKeySecret
    endpoint=oss-cn-beijing.aliyuncs.com
    accessKeyId=your accessKeyId
    accessKeySecret=your accessKeySecret
    #接受阿里云上传回调的主机ip和端口，该主机需要外网能直接访问（直传功能）
    callbackTarget=http://localhost
    #处理上传回调的servlet截取的地址，直传成功后阿里云oss将会对上面配置的主机地址发送url为改配置url的post请求（直传功能）
    callbackUrl=/oss
    #需要上传回调所带的参数，上面配置的的请求将会携带callbackBody配置的参数（直传功能）
    callbackBody=filename=${object}&bucket=${bucket}&size=${size}

在工程上配置属性文件时，可以根据使用的存储类型，配置需要的部分。



## 功能调用 ##

###1.阿里云oss###


**非直传**

组件提供的API，操作阿里云oss，所有的api都通过FileManager类静态调用，例子：FileManager.uploadFile("yourbucket","test.txt",content)。
    
	//上传，参数（bucket名，文件名，上传文件字节数组），返回（上传文件的文件名）
    String uploadFile(String bucketName, String fileName,byte[] fileContent)

 	//下载，参数（bucket名，文件名），返回（下载文件的字节数组）
 	byte[] downLoadFile(String bucketName,String fileName) ;

	//删除，参数（bucket名，文件名），返回（删除是否成功标志）
    boolean deleteFile(String bucketName,String fileName)

    //获取文件url，参数（bucket名，文件名，过期时间），返回（上传文件的url）目前expired参数只支持阿里云oss私有bucket，其他系统可置为0
	String getUrl(String bucketName, String fileName,int expired)
	
	//获取图片url，参数（bucket名，文件名，过期时间），返回（图片的url）与getUrl的区别是文件名可以使用类似example.jpg@100h_100w这种格式产生略缩图，）目前expired参数只支持阿里云oss私有bucket，其他系统可置为0
	String getImgUrl(String bucketName,String fileName,int expired)
	
	

example测试类


	/**
	 * 阿里云上传下载删除测试
	 * 测试前请设置application.properties的storeType=AliOss
	 * 参数"your bucket"为阿里云oss的bucket名，请更换为您的bucket名
	 * @throws Exception
	 */
	@Test
	public void testAliUpload() throws Exception {
		String filename;
		//借用文件流生成文件的二进制数组
		LocalClient client =LocalClient.getInstance();
		//参数为测试文件路径;		
		byte[] content =client.download("/etc/filetest/test.txt");
		//==================准备工作完毕=============================
		
		//阿里云上传
		filename=FileManager.uploadFile("your bucket","test.txt",content);
		//阿里云下载
		byte[] downloadContent=FileManager.downLoadFile("your bucket",filename);
		//获取文件url
		String url = FileManager.getUrl("your bucket", filename, 60);
		//获取图片url(将上传文件路径改为图片)
		//String imgurl = FileManager.getImgUrl(Private, filename, 60);
		//删除
		boolean flag=FileManager.deleteFile("your bucket",filename);
		
		System.out.println(filename);
		System.out.println(url);
		//System.out.println(imgurl);
		System.out.println("删除状态"+flag);
		//Assert.isTrue(flag);
	}	

**直传**

直传功能目前只支持阿里云oss，该功能使文件上传请求和流量不再经过应用服务器而直接交给oss处理，直传功能由阿里云官方文档推荐的框架搭建(相关连接：<a href="https://help.aliyun.com/document_detail/31927.html?spm=5176.doc31923.2.3.VjMHHN">web端直传</a>)

流程

用户请求->Servlet获取签名->js装配获得的签名并上传文件->oss上传文件后向Servlet发送回调->Servlet对oss回调请求处理后向oss发送处理结果->oss向用户返回上传结果。

首先建立一个Servlet类继承CallbackServer，通过该Servlet装配oss直传需要的签名数据，并处理oss只直传完毕后的回调
    
    package com.yonyou.iuap.file;
    
    import java.io.IOException;
    import java.util.Map;
    
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    @WebServlet(asyncSupported = true,name = "Oss",urlPatterns = { "/oss" })
    public class OssDirectServlet extends CallbackServer {
    	
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = 1L;
    
    	protected void doPost(HttpServletRequest request, HttpServletResponse response)
    			throws ServletException, IOException {
    		String ossCallbackBody = GetPostBody(request.getInputStream(), Integer.parseInt(request.getHeader("content-length")));//获取回调
    		boolean ret = VerifyOSSCallbackRequest(request, ossCallbackBody);//验证回调
    		
    		if (ret)
    		{
    			//验证通过进行操作
    			Map<String, Object> map = getUrlParams(ossCallbackBody);
    			String filename=(String) map.get("filename");//获取存储的文件名
    			String bucket=(String) map.get("bucket");//获取存储的bucket
    			System.out.println(filename);
    			System.out.println(bucket);
    			
    			response(request, response, "{\"Status\":\"OK\"}", HttpServletResponse.SC_OK);
    
    		}
    		else
    		{
    			response(request, response, "{\"Status\":\"verdify not ok\"}", HttpServletResponse.SC_BAD_REQUEST);
    		}
    	}
    
    }

    
通过js进行上传

    
    function init(){
    
    //提交
    //获取oss签名参数
    var ret = get_signature()
    	//装配发送到oss文件上传url   
    var formData = new FormData(); 
    var file=$("#file").prop('files');
       
    var filepath=$("#file").val();
    var arr=filepath.split('\\');
    var filename=arr[arr.length-1];//===================获得本地文件名（需要你提供）
    
    if (ret == true)
    {
    	//装配oss签名参数
    	formData.append("name",filename);
    	formData.append("key", key);
    	formData.append("policy", policyBase64);
    	formData.append("OSSAccessKeyId", accessid);
    	formData.append("success_action_status", '200');
    	formData.append("callback", callbackbody);
    	formData.append("signature", signature);
    	formData.append("file",document.getElementById('file').files[0])//===================获得文件数据（需要你提供）
    }
    //发送文件上传请求   
    	$.ajax({  
    		url: host,  
    		type: 'POST',  
    		data: formData,  
    		async: false,  
    		contentType: false, //必须
    		processData: false, //必须
    		success: function (returndata) {  
   	 			alert(returndata);  
    		},  
   			error: function (returndata) {  
    			alert(returndata);  
    		}  
    	});  
  	}
    
    //发送到应用服务器 获取oss签名参数
    function send_request(){
    	var obj
    	$.ajax({
    		type : 'GET',
    		async : false,
    		url :'http://localhost/example_iuap_file/oss?bucketname='+document.getElementById('bucketname').value,//==========租户id（需要你提供）
    		success : function(data){
    		obj=$.parseJSON(data);
    		} 
    	});
    	return obj;
    }
    
    //获取到签名参数后将参数填入变量待用
    function get_signature()
    {
    	//可以判断当前expire是否超过了当前时间,如果超过了当前时间,就重新取一下.3s 做为缓冲
    	expire = 0
    	now = timestamp = Date.parse(new Date()) / 1000; 
    	console.log('get_signature ...');
    	console.log('expire:' + expire.toString());
    	console.log('now:', + now.toString())
    	if (expire < now + 3){
    		console.log('get new sign')
    		var obj = send_request(obj)
    		policyBase64 = obj['policy']
   			accessid = obj['accessid']
    		signature = obj['signature']
    		expire = parseInt(obj['expire'])
    		callbackbody = obj['callback'] 
    		host =obj['host']
    		key = obj['perfix']+'${filename}'
    		return true;
    	}
    	return false;
    };
    
    $("#submit").attr("onclick","init();");


发送上传请求后会在上文中的Servlet类dopost方法处理oss回调，最终oss将结果发送回前台js的ajax回调。

**临时文件url与永久文件url**


oss文件系统能够通过getUrl方法返回文件的url，返回的url有临时和永久两种。该接口如下


public static String getUrl(BucketPermission permission,String fileName,int expired) 

三个参数为ossbucket的权限，文件名，过期时间。其中permission的值可以为BucketPermission.private、BucketPermission.read、BucketPermission.full。如果需要临时url的话permission值需要为private，并且传入expired值为过期时长。如果需要永久的文件url前提是上传文件bucket需要为read或full权限（不推荐使用），设置bucket权限可以在oss控制台(相关连接：<a href="https://help.aliyun.com/document_detail/31885.html?spm=5176.doc31886.6.101.vSWwVY">建立新的bucket</a>)，选择需要修改权限的bucket，Bucket属性标签有读写权限修改。在geturl时参数传入BucketPermission.read或者BucketPermission.full，expired可以为0。

**略缩图功能**

首先在oss控制台，选择存储略缩图文件的bucket，在左边的菜单中选择图片处理，选择开通图片处理。
然后通过下面的接口可以返回略缩图url：

public static String getImgUrl(BucketPermission permission,String fileName,int expired)

略缩图也是分为临时与永久url，方法与上文相同。通过该方法获得url以后附加类似@100h的参数产生略缩图。

例子：

将图缩略成高度为100，宽度按比例处理。

http://image-demo.img-cn-hangzhou.aliyuncs.com/example.jpg@100h

将图缩略成宽度为100，高度按比例处理。

http://image-demo.img-cn-hangzhou.aliyuncs.com/example.jpg@100w

###2.FastDFS###


组件提供API，操作FastDFS

FastDfs API与使用oss系统时是相同的，所有的api都通过FileManager类静态调用，例子：FileManager.uploadFile("private","test.txt",content)。

在使用FastDfs模式时，bucketName参数代表文件在操作FastDFS系统存储的权限（private、read、full）该权限将存入FastDFS文件的metadata中，目前组件还没有对不同权限的文件进行访问限制处理，请等待以后的更新。FastDfs模式下，bucketName目前为三个固定值PRIVATE、READ、FULL，代表三种权限。

	//上传，参数（bucket名，文件名，上传文件字节数组），返回（上传文件的文件名）
    String uploadFile(String bucketName, String fileName,byte[] fileContent)
	
 	//下载，参数（bucket名，文件名），返回（下载文件的字节数组）
 	byte[] downLoadFile(String bucketName,String fileName) ;
	
	//删除，参数（bucket名，文件名），返回（删除是否成功标志）
    boolean deleteFile(String bucketName,String fileName)

    //获取文件url，参数（bucket名，文件名，过期时间），返回（上传文件的url）目前expired参数只支持阿里云oss私有bucket，其他系统可置为0
	String getUrl(String bucketName, String fileName,int expired)
	
	//获取图片url，参数（bucket名，文件名，过期时间），返回（图片的url）与getUrl的区别是文件名可以使用类似example.jpg@100h_100w这种格式产生略缩图，）目前expired参数只支持阿里云oss私有bucket，其他系统可置为0
	String getImgUrl(String bucketName,String fileName,int expired)

example测试类
    
    /**
	 * fastdfs上传下载删除测试
	 * 测试前请设置application.properties的storeType=FastDfs
	 * @throws Exception
	 */
	@Test
	public void testFdfsUpload() throws Exception {
		String filename;
		//借用文件流生成文件的二进制数组
		LocalClient client =LocalClient.getInstance();		
		//参数为测试文件路径
		byte[] content =client.download("/etc/filetest/test.txt");
		//==================准备工作完毕=============================
		
		//fastdfs上传
		filename=FileManager.uploadFile(Private, "test.txt",content);
		//fastdfs下载
		byte[] downloadContent=FileManager.downLoadFile(Private,filename);
		//获取文件url
		String url = FileManager.getUrl(Private, filename, 0);
		//获取图片url(将上传文件路径改为图片)
		//String imgurl = FileManager.getImgUrl(Private, filename, 0);
		//删除
		boolean flag=FileManager.deleteFile(Private,filename);	
		
		
		System.out.println(filename);
		System.out.println(url);
		//System.out.println(imgurl);
		System.out.println("删除状态"+flag);
		Assert.isTrue(flag);
	}

**略缩图功能**

fdfs使用略缩图功能时 需要搭建nginx提供文件访问服务

安装nginx
    
    1：安装前准备

	yum -y install zlib zlib-devel openssl openssl-devel pcre-devel gcc gcc-c++ autoconf automake gd-devel

	groupadd -r nginx

	useradd -s /sbin/nologin -g nginx -r nginx

	mkdir /var/tmp/nginx/client -pv

	touch /var/lock/nginx.lock

	2：nginx配置和安装关键步骤示例（仅供参考）

	cd /usr/local

	wget http://nginx.org/download/nginx-1.8.0.tar.gz

	tar -zxvf nginx-1.8.0.tar.gz

	cd nginx-1.8.0


	./configure \
	--prefix=/usr \
	--sbin-path=/usr/sbin/nginx \
	--conf-path=/etc/nginx/nginx.conf \
	--error-log-path=/media/disk1/nginx/logs/error.log \
	--pid-path=/var/run/nginx/nginx.pid \
	--lock-path=/var/lock/nginx.lock \
	--user=nginx \
	--group=nginx \
	--with-pcre=/usr/local/src/pcre-8.35 \
	--with-http_ssl_module \
	--with-http_flv_module \
	--with-http_gzip_static_module \
	--http-log-path=/media/disk1/nginx/logs/access.log \
	--http-client-body-temp-path=/media/disk1/nginx/client \
	--http-proxy-temp-path=/media/disk1/nginx/proxy \
	--http-fastcgi-temp-path=/media/disk1/nginx/fcgi \
	--with-http_stub_status_module \
	--with-poll_module \
	--with-http_realip_module \
	--add-module=/usr/local/src/ngx_cache_purge-2.3 \
	--add-module=/usr/local/src/fastdfs-nginx-module/src \
	--with-http_image_filter_module \
	--with-cc-opt=-Wno-error 

	nginx configure时需要注意带上fastdfs-nginx-module、http_image_filter_module等模块

	3：nginx安装检查

	检查是否启动
	ps -ef |grep nginx

	启动
	nginx -c /etc/nginx/nginx.conf

	查看80端口是否被监听
	netstat -tulnp |grep :80

4: nginx 服务配置

	将nginx文件复制到/etc/init.d下并修改权限

	chmod +x nginx

5：配置nginx.conf
	
    location /g1/M00 {
	    root /data/fdfs;
	    ngx_fastdfs_module;
    }

    location  ~* /g1/M00/(.*)@(.*)$ {
    	set $img_name $1;
    	set $img_param $2;
    	
    	if ( $img_param ~* "([\d]+)h_([\d]+)w" ){
    	set $img_height $1;
    		set $img_width $2;
    	}
    	
    	if ( $img_param ~* "([\d]+)h" ){
    	set $img_height $1;
    		set $img_width -;
    	}
    	
    	if ( $img_param ~* "([\d]+)w" ){
    	set $img_height -;
    		set $img_width $1;
    	}
    	
	    root /data/fdfs;
	    ngx_fastdfs_module;
	    
	    rewrite /g1/M00/(.*)@(.*)$ /g1/M00/$1 break;
	    	
	    image_filter resize $img_width $img_height;
	    image_filter_buffer 5M;
	    image_filter_jpeg_quality 80;
	    image_filter_transparency on;
	    
	    proxy_cache content_image;
	    proxy_cache_key $request_uri$is_args$args;
	    proxy_cache_valid 200 304 12h;
	    expires 30d;
    }

6：	配置fdfs_auth.conf

	拷贝iuap-file组件jar包lib文件夹下的fdfs_auth.conf文件至/etc/fdfs/

7：配置mod_fastdfs.conf
	
	cp /usr/local/src/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/
	
	修改配置项
	tracker_server=tracker服务器io:22122#修改为tracker server信息
	url_have_group_name= true #改为true
	store_path0=/data/fdfs #改为数据存储路径，和storage.conf一样

8:拷贝http.conf和mime.types

	这两个文件在FastDFS的源码包里有（FastDFS_v5.05.tar.gz）
	cp /usr/local/src/FastDFS/conf/http.conf /etc/fdfs/
	cp /usr/local/src/FastDFS/conf/mime.types /etc/fdfs/
	


5：服务启动

	服务方式启动
	service nginx start

	停止
	service nginx stop

	重新加载
	service nginx reload

**接口调用**

然后通过下面的接口可以返回略缩图url：

public static String getImgUrl(BucketPermission permission,String fileName,int expired)

fileName后面附加类似@100h的参数产生略缩图。

例子：

将图缩略成高度为100，宽度按比例处理。

example.jpg@100h

将图缩略成宽度为100，高度按比例处理。

example.jpg@100w

将图缩略成高度为100，宽度为100。
example.jpg@100h_100w


###3.本地文件系统###

本地文件系统一般用于开发测试，不建议部署为生产系统

组件提供的API，操作本地文件系统


    //上传，参数（bucket名传null，文件名，上传文件字节数组），返回（上传文件的文件名）
    filename=FileManager.uploadFile(null,"test.txt", content);
	//下载，参数（bucket名传null，文件名），返回（下载文件的字节数组）
    byte[] content =client.download("/etc/filetest/test.txt");
    //删除，参数（bucket名传null，文件名），返回（删除是否成功标志）
    boolean flag=FileManager.deleteFile(null,filename);


example测试类


	/**
	 * 文件系统上传下载删除测试
	 * 测试前请设置application.properties的storeType=Local
	 * @throws Exception
	 */
	@Test
	public void testFSUpload() throws Exception {
		String filename;
		//下载并将文件转换为二进制数组
		//借用文件流生成文件的二进制数组
		LocalClient client =LocalClient.getInstance();		
		byte[] content =client.download("/etc/filetest/test.txt");
		
		//上传	
		filename=FileManager.uploadFile(null,"test.txt", content);
		//删除
		boolean flag=FileManager.deleteFile(null,filename);
		
		System.out.println(filename);
		System.out.println("删除状态"+flag);
		Assert.isTrue(flag);
	}


**fdfs权限验证**

1.简介

fdfs权限模块是用来解决fdfs文件的权限验证问题的nginx模块，在用户对具有权限的的fdfs文件发出访问请求时，该模块发出鉴权子请求，用户来实现拦截该请求的controller或者servlet，根据用户请求的cookies、http参数等信息判断该用户是否具有访问该文件的权限，具有权限则放行请求，不具有权限则返回401未授权错误。

2.部署结构


如下图所示通过一个主nginx管理fdfs文件系统，在主nginx上部署fdfs权限模块就可以对fdfs文件的访问作权限控制了，再鉴权的同时也不会影响文件缓存。用户请求发给主nginx，由主nginx的fdfs权限模块判断该请求是否具有权限，如果有权限则发给fdfs tracker处理，tracker再从storage处取得图片。

<img src="/images/fdfs-auth1.jpg"/>




请求具体流程，其中文件鉴权服务器可以为webapp的controller或者servlet（可以参见示例工程example_iuap_file的FdfsAuthController类）。

<img src="/images/fdfs-auth2.jpg"/>


3.安装

搭建fdfs文件验证分为两个部分，安装带有fdfs文件权限模块的nginx，以及部署负责验证文件权限的webapp。nginx模块负责发送文件鉴权子请求，webapp收到该子请求进行权限鉴定。


nginx安装


	1：确保依赖安装
	yum install –y openssl-devel pcre-devel zlib-devel

    2：下载相关依赖压缩包,并解压到/usr/local/src下，（包括fastdfs-nginx-module_v1.16、pcre-8.35、ngx_cache_purge-2.3、ngx_fdfs_auth_module，其中ngx_fdfs_auth_module模块源码文件在iuap-file组件jar包内的lib文件夹中，将lib\ngx_fdfs_auth_module拷贝到/usr/local/src即可）

    3：编译和安装nginx

	创建用户和响应的目录和文件，请参考nginx的安装文档

	cd /usr/local/src/nginx-1.8.0

	./configure \
	--prefix=/usr \
	--sbin-path=/usr/sbin/nginx \
	--conf-path=/etc/nginx/nginx.conf \
	--error-log-path=/media/disk1/nginx/logs/error.log \
	--pid-path=/var/run/nginx/nginx.pid \
	--lock-path=/var/lock/nginx.lock \
	--user=nginx \
	--group=nginx \
	--with-pcre=/usr/local/src/pcre-8.35 \
	--with-http_ssl_module \
	--with-http_flv_module \
	--with-http_gzip_static_module \
	--http-log-path=/media/disk1/nginx/logs/access.log \
	--http-client-body-temp-path=/media/disk1/nginx/client \
	--http-proxy-temp-path=/media/disk1/nginx/proxy \
	--http-fastcgi-temp-path=/media/disk1/nginx/fcgi \
	--with-http_stub_status_module \
	--with-poll_module \
	--with-http_realip_module \
	--add-module=/usr/local/src/ngx_cache_purge-2.3 \
	--add-module=/usr/local/src/fastdfs-nginx-module/src \
	--add-module=/usr/local/src/ngx_fdfs_auth_module \
	--with-http_image_filter_module \
	--with-cc-opt=-Wno-error

    4：设置启动文件和nginx配置文件

	请参考安装目录下的tracker下的nginx和nginx.conf

	修改对应的ip后分别拷贝到/etc/init.d/、/etc/nginx/

    5: 启动nginx

        service nginx start

    6: 查看启动进程

        ps aux | grep nginx


fdfs权限模块配置文件/etc/fdfs/fdfs_auth.conf，该文件决定了鉴权子请求将会发送到哪个uri

	[fdfs_auth]
	auth_path=/fdfsauth




配置文件nginx.conf，加入以下内容

			#/g1/m00截取fdfs文件请求
            location /g1/M00 {
				#权限模块开关
				fdfs_auth open;
                proxy_pass http://fdfs_g1;
            }
			#鉴权子请求配置/fdfsauth为fdfs权限模块配置文件中设置的地址。
			 location /fdfsauth {
  				internal;#指明这个目录不能在外部直接访问到
				#proxy_pass指向拦截鉴权子请求的webapp鉴权controller
				proxy_pass http://172.20.6.48:8090/example_iuap_file/fdfsauth;
			}


鉴权webapp，这部分可以使工程中的一个controller或者是servlet，它接受nginx发送的鉴权子请求。并且该子请求携带了原请求的cookies、http参数等信息。通过这些信息确定该用户是否具有请求文件的权限。（可以参见示例工程example_iuap_file的FdfsAuthController类）。

当该controller返回http请求正常时，用户请求才能被允许。出现异常状态则被视为鉴权未通过。


3.性能扩展

可以通过部署多台鉴权服务器的方式，来提高系统性能

<img src="/images/fdfs-auth3.jpg"/>




##常用接口##
**文件组件api接口介绍**

组件接口类FileManager

FastDfs模式下，bucketName为三个固定值PRIVATE、READ、FULL，代表三种权限。

<table style="border-collapse:collapse">
	<tr>
		<th>方法名</th>
		<th>参数</th>
		<th>返回值</th>
		<th>功能说明</th>
	</tr>

	<tr>
		<td>uploadFile</td>
		<td>
			1. String bucketName（bucket名）<br/>
			2. String fileName（上传文件名）<br/>
			3. byte[] fileContent（上传文件字节数组）<br/>
		</td>
		<td>String（上传后的文件名）</td>
		<td>上传文件</td>
	</tr>
	<tr>
		<td>downLoadFile</td>
		<td>
			1. String bucketName（下载文件所在bucket名）<br/>
			2. String fileName（下载文件名）<br/>
		</td>
		<td>byte[]（下载文件的二进制数组）</td>
		<td>下载文件</td>
	</tr>
	<tr>
		<td>deleteFile</td>
		<td>
			1. String bucketName（bucket名）<br/>
			2. String fileName（要删除的文件名）<br/>
		</td>
		<td>boolean（删除文件是否成功）</td>
		<td>删除文件</td>
	</tr>
	<tr>
		<td>getUrl</td>
		<td>
			1. String bucketName(bucket名)<br/>
			2.  String fileName（获取url的文件名<br/>
			3.  int expired（单位 秒 ，连接过期时间,目前该参数只支持阿里云oss私有bucket)<br/>
		</td>
		<td>String（文件的url）</td>
		<td>返回文件url</td>
	</tr>
	<tr>
		<td>getImgUrl</td>
		<td>
			1. String bucketName(bucket名)<br/>
			2. String fileName（获取url的文件名）<br/>
			3. int expired（单位 秒 ，连接过期时间,目前该参数只支持阿里云oss私有bucket）<br/>
		</td>
		<td>String（图片的url）</td>
		<td>
			返回图片url <br/>
			使用阿里云oss时在fileName加入类似@100h的参数能生成略缩图<br/>
			使用fdfs时在fileName加入类似@100h的参数能生成略缩图（需要布置nginx支持）<br/>
		</td>
	</tr>
</table>
	
**更多详细的使用方法，请参考示例工程(DevTool/examples/example_iuap_file)**