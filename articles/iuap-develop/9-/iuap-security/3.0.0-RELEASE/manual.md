# iuap-security组件简介 #

## 业务需求
能够提供安全访问时候请求的签名、验证，传输,简单数据加密解密，编码解码等功能，包含对Hmac、RSA等算法的封装，方便业务开发使用。

## 解决方案

iuap-security组件引入OWASP组织的开源项目ESAPI，利用ESAPI，平台实现了对跨站脚本攻击（XSS），sql脚本注入等安全问题的预防。 ESAPI具有一个安全接口集，其对每一种安全控制有一种参考实现并支持自定义实现。组件在ESAPI的基础上进行扩展。加入了对HTML、CSS、JavaScript、XML、URL等编码与解码，信息的加密解密，信息的摘要签名，防sql注入等功能的API。

iuap-security组件通过可配置的方式提供ESAPI给开发人员使用，方便了业务开发人员使用，iuap-security提供了rest api的签名和验签，在调用端和服务端分别做信息的摘要和验证，确保数据的完整性，防止恶意篡改。

## 功能说明 
1. RestAPI签名和验签
2. 编码和解码
3. 加密和解密

# 整体设计

## 依赖环境

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：

    <dependency>
      <groupId>com.yonyou.iuap</groupId>
      <artifactId>iuap-security</artifactId>
      <version>${iuap.modules.version}</version>
    </dependency>
	    
${iuap.modules.version} 为平台在maven私服上发布的组件的version。


# 使用说明

## 属性文件配置 ##

application.properties配置示例：
	
	#RSA数字签名算法，目前JDK支持MD2withRSA,MD5withRSA,SHA1withRSA,都是1024bits
	UAP.DigitalSignatureAlgorithm=SHA1withRSA
	
	#RSA秘钥长度1024/2048
	UAP.DigitalSignatureKeyLength=1024
	
	#随机算法
	UAP.RandomAlgorithm=SHA1PRNG
	
	#HMAC摘要算法，目前jdk支持：HmacSHA1 (160 bits), HmacSHA256 (256 bits), HmacSHA384 (384 bits),HmacSHA512 (512 bits).
	UAP.KDF.PRF=HmacSHA1
	
	#签名或者摘要算法目前支持HMAC、RSA.
	UAP.AUTH.ALG=HMAC
	
	#客户端身份文件路径
	bpm.client.credential.path=/etc/authfile_bpm.txt
	
	#如果客户端需要用多个服务提供的证书，可以通过appid来区分不同的证书目录
	#appid.client.credential.path=/etc/authfile_appid.txt
	  
示例工程中的antisamy-esapi.xml、ESAPI.properties、validation.properties在jar包中已经包含，如果没有需要覆盖的属性，则不需要添加。ESAPI.properties中如果需要覆盖Encryptor.MasterKey和Encryptor.MasterSalt的值，请参考示例工程中的setMasterKey.bat，生成新值，并覆盖到classpath中的ESAPI.properties中。

## API接口 ##
### RestAPI签名 ###

在请求端，根据URL参数（含ts值）、请求端IP地址、表单参数以及Content-Length的大小进行生成签名sign。

	SignProp prop = SignPropGenerator.genSignProp(url);
	signProp.setIp(ip);

	//post请求时需要
	signProp.setPostParamsStr(PostParamsHelper.genParamsStrByMap(parameter));
	signProp.setContentLength(ContentLength);
	
	String sign = ClientSignFactory.getSigner(cert).sign(prop);

注意：如果配置多份客户端证书，需要在getSinger的时候指定前缀,示例如下：

	ClientSignFactory.getSigner("bpm")


**SignProp中包含的参数说明**

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>参数名</th>
			<th>必选</th>
			<th>类型</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>ts</td>
			<td>true</td>
			<td>字符串</td>
			<td>请求发出的时间戳，在请求的url中</td>
		</tr>
		<tr>
			<td>ip</td>
			<td>false</td>
			<td>请求端IP地址</td>
			<td>请求端IP地址</td>
		</tr>
		<tr>
			<td>postParamsStr</td>
			<td>false</td>
			<td>字符串</td>
			<td>POST表单参数Map生成的字符串，可以调用PostParamsHelper中的方法生成</td>
		</tr>
		<tr>
			<td>contentLength</td>
			<td>false</td>
			<td>字符串</td>
			<td>HttpServletRequest中取得的Content-Length的值</td>
		</tr>
	</tbody>
</table>

**返回值**

sign 生成的验证签名,类型为字符串。


### RestAPI验签 ###

**描述**

在服务端，首先比较当前时间ts’与ts的大小，如果其差值不大于DEFAULT_EXPIRED，则从HTTP请求中获得时间戳ts、URL参数、请求的来源IP地址、表单参数以及ContentLength的大小生成sign’。此时如果sign与sign’相同，则签名验证通过

	SignProp prop = SignPropGenerator.genSignProp(url);

	//服务端需要获取客户端的真实ip
	signProp.setIp(ip);

	//post请求时需要
	signProp.setPostParamsStr(PostParamsHelper.genParamsStrByMap(request));
	//在服务端获取请求的contentLength
	signProp.setContentLength(contentLength);

	DemoServerVirifyFactory factory = new DemoServerVirifyFactory();
	Boolean result = factory.getVerifier(appid).verify(sign, prop);

DemoServerVirifyFactory为开发者实现的类，需要实现获取证书的方法。


**参数说明**

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>参数名</th>
			<th>类型</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>appid</td>
			<td>字符串</td>
			<td>服务端存储的证书中对应的客户端的唯一标识</td>
		</tr>
		<tr>
			<td>sign</td>
			<td>字符串</td>
			<td>请求中传入的sign，用来与服务端生成的sign进行对比</td>
		</tr>
		<tr>
			<td>prop</td>
			<td>SignProp</td>
			<td>需要验证签名的参数集合对象</td>
		</tr>
	</tbody>
</table>


**返回值**

验证是否成功，类型为boolean。

### 编码 ###

**描述**

数据编码分为多种形式，如html编码、javascript编码、css编码、sql编码等，它们主要是用来解决各类注入问题：xss注入、sql注入。除此之外，还有Base64编码，常用来将非ASCII字符的数据转换成ASCII字符

请求方法

- html编码

		String safe =IUAPESAPI.encoder().htmlEncode( input);

- 属性编码
 
		String safe =IUAPESAPI.encoder().htmlAttributeEncode (input);
	
- js编码

		String safe =IUAPESAPI.encoder().javaScriptEncode ( input);
- url编码

		String safe =IUAPESAPI.encoder().urlEncode (input);

- css编码

		String safe =IUAPESAPI.encoder().cssEncode ( input);

- sql注入示例

		String queryCondition = IUAPESAPI.encoder().sqlPreparedString( input,paras, DatabaseCodec.ORACLE);

- base64

		String base64Text = IUAPESAPI.encoder().encodeForBase64(input);


**请求参数说明**

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>参数名</th>
			<th>类型</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>input</td>
			<td>字符串</td>
			<td>要编码的内容</td>
		</tr>
	</tbody>
</table>

**返回值**

编码后的字符串。


### 对称加密算法 ###

**功能描述**

对称加密算法可以用作存储加密，即将数据持久化前对数据进行加密。

#### 加密 ####

请求方法

	String safe = IUAPESAPI.encryptor().encrypt(input);


参数说明


<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>参数名</th>
			<th>类型</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>input</td>
			<td>字符串</td>
			<td>要加密的内容</td>
		</tr>
	</tbody>
</table>


返回值

加密后的字符串。


#### 解密 ####

请求方法

	String safe = IUAPESAPI.encryptor(). decrypt(input);


参数说明

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>参数名</th>
			<th>类型</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>input</td>
			<td>字符串</td>
			<td>要解密的内容</td>
		</tr>
	</tbody>
</table>

返回值

解密后的字符串。


### 非对称加密算法 ###

描述

非对称加密算法可以用作传输加密，即数据从请求端发送到服务端时，对敏感数据用公钥进行加密，在服务端用私钥进行解密，这里我们提供了RSAUtils工具类进行RSA加密。

#### 加密 ####

请求方法

	//首次生成公钥的示例
	KeyPair gk = RSAUtils.generateKeyPair();		
	RSAPublicKey publicKey = RSAUtils.getDefaultPublicKey();

	//调用方法进行加密
	byte[] encryptBytes = RSAUtils.encrypt(publicKey, input.getBytes());

参数说明

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>参数名</th>
			<th>类型</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>publicKey</td>
			<td>RSAPublicKey</td>
			<td>公钥</td>
		</tr>
		<tr>
			<td>input</td>
			<td>byte[]</td>
			<td>要解密的内容的字节数组</td>
		</tr>
	</tbody>
</table>


返回值

加密后的二进制数组。


#### 解密 ####

请求方法
	
	RSAPrivateKey privateKey = RSAUtils.getDefaultPrivateKey();

	byte[] data = RSAUtils.decrypt(privateKey, encryptBytes);


参数说明

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>参数名</th>
			<th>类型</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>privateKey</td>
			<td>RSAPrivateKey</td>
			<td>私钥</td>
		</tr>
		<tr>
			<td>encryptBytes</td>
			<td>byte[]</td>
			<td>要解密的内容的字节数组</td>
		</tr>
	</tbody>
</table>



返回值

解密后的内容，类型为字节数组。


更加详细的配置和示例请参考示例工程(DevTool/examples/example\_iuap_security)