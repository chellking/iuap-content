**文件组件api接口介绍**

组件接口类FileManager


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