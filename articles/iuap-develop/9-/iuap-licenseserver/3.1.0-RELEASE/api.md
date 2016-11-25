
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
