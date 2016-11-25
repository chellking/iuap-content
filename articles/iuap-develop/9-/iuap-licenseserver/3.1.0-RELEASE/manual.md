#LicenseServer组件概述#

## 业务需求 ##

为应对各业务场景下授权机制的要求, 开发LicenseServer项目, 避免每个项目开发一套, 使各项目能够公用, 也可以根据需要单独部署.


## 解决方案 ##

iUAP平台采用LicenseServer作为License授权机制，实现对各业务场景下的License授权及管理。

本组件提供对"CPU个数"场景接口、"特征数"场景、""用户数"场景授权管理接口的封装，供其它产品或者项目简便的集成和管理授权。

iUAP LicenseServer组件实现了Http连接池和LicenseClient进行交互，使用非对称加密算法保证传输的安全性。 分布式架构保证了服务的高可用性，水平无限制集群扩展，持续提供可用服务。 同时，LicenseServer提供LicenseClient的maven构件方便基于maven的集成。


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-licenseclient</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 部署结构 ##

iuap-licenseclient组件使用用java语言通过HttpClient方式进行连接，建立连接池并支持安全加密访问。

iuap-licenseserver支持1到N台的水平扩展部署, 示意图如下： 

<img src="/images/licenseserver_ha.jpg"/>

# 使用说明 #

## 组件包说明 ##

iuap-client组件利用ApacheHttpClient客户端，通过非对称加密机制，提供安全方式的Http连接池交互。

## 组件配置 ##

**1:在classpath下的iuap-licenseclient-conf.properties属性文件中，配置iuap-licenseserver的目标url**

	!license-server 的远程地址的配置
	license-server-url=http://licenseserver.com/iuap-licenseserver/client



**2:工程中引入对iuap-licenseclient组件的依赖**

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-licenseclient</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>

${iuap.modules.version}为在pom.xml中定义的引用iuap-licenseclient的版本。

**4:代码中调用组件提供的API，操作licenseclient**

	/**
	 * 根据产品编码注册license, 返回值为下次和服务器进行校验的时间(毫秒为单位), -1代表验证失败.
	 */
	@Test
	public void testRegLicense() {
		String productCode = "abc"; // 产品编码
		long ret = CPULicenseService.regLicense(productCode);
		System.out.println(ret + "  " + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date(ret)));
	}


**5:更多API操作和配置方式，请参考缓存对应的示例工程(DevTool/examples/example\_iuap_licenseserver)**

## 工程样例 ##

开发工具包DevTool中携带了对LicenseClient组件的示例工程，位置位于DevTool/examples/example\_iuap\_licenseserver下，在IUAP_STUDIO中导入已有的Maven工程，可以将示例工程导入到工作区。示例工程中有较为完整的对iuap-client组件的使用示例代码。
<img src="/images/prj-snapshot.png"/>

## 开发步骤 ##

- 新建iuap-licenseclient-conf.properties属性文件, 内容配置如下:

		!license-server 的远程地址的配置
		license-server-url=http://licenseserver.com/iuap-licenseserver/client

- 属性文件中配置的license-server-url应该是服务器的licenseServer对接口提供的地址.

- 参考测试用例, 使用相应服务类的静态方法直接调用即可.


## 常用接口 ##


- CPULicenseService -- "CPU个数"场景接口

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
			<td>regLicense(String productCode)</td>
			<td>String productCode（产品编码）</td>
			<td>long</td>
			<td>根据产品编码注册license; <br>返回值为 下次和服务器进行校验的时间(毫秒为单位), -1代表验证失败.</td>
		</tr>
	</tbody>
</table>


- FeatureCountService -- "特征数"场景

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
			<td>getPermitCount(String productCode)</td>
			<td>String productCode（产品编码）</td>
			<td>int</td>
			<td>根据productCode获取此产品可分配的license数量.</td>
		</tr>
	</tbody>
</table>


- UserCountService -- "用户数"场景

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
			<td>regLicense(String identifier, String hostName, String productCode)</td>
			<td>String identifier（客户端唯一标识）, <br>String hostName(主机名), <br>String productCode(产品编码)</td>
			<td>int</td>
			<td>根据客户端唯一标识, 计算机名称, 产品编码 进行注册license操作.<br>返回值为 下次和服务器进行校验的时间(毫秒为单位), -1代表验证失败.</td>
		</tr>
		<tr>
			<td>cancelLicense(String identifier, String productCode)</td>
			<td>String identifier（客户端唯一标识）, String productCode(产品编码)</td>
			<td>boolean</td>
			<td>根据用户id/客户端唯一标识 和 产品编码 进行 注销license操作, 返回值为是否操作成功.</td>
		</tr>
		<tr>
			<td>getRemainLicenseCount(String productCode)</td>
			<td>String productCode(产品编码)</td>
			<td>int</td>
			<td>根据产品编码获取剩余license数量.<br>返回值为剩余license数量. -1代表请求异常. 大于等于0为正常返回</td>
		</tr>
		<tr>
			<td>isAvaliableLicense(String productCode)</td>
			<td>String productCode(产品编码)</td>
			<td>boolean</td>
			<td>根据产品编码获取是否有license可用, 返回值true为有可用license, false为没有可用license.</td>
		</tr>
	</tbody>
</table>
