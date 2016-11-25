
- CacheManager

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
			<td>set(final String key, final T value)</td>
			<td>String key（缓存key）, T value（缓存对象，允许序列化）</td>
			<td>void</td>
			<td>放置键值对对应的缓存</td>
		</tr>
		<tr>
			<td>setex(final String key, final T value，final int timeout)</td>
			<td>String key（缓存key）, T value（缓存对象，允许序列化），int timeout（缓存的有效期）</td>
			<td>void</td>
			<td>放置键值对对应的缓存并设置缓存有效期，单位为秒</td>
		</tr>
		<tr>
			<td>expire(final String key, final int timeout)</td>
			<td>final String key（缓存key）, final int timeout（缓存的有效期）</td>
			<td>void</td>
			<td>修改缓存信息的有效期</td>
		</tr>
		<tr>
			<td>exists(final String key)</td>
			<td>final String key（缓存key）</td>
			<td>Boolean（是否存在的标志）</td>
			<td>判断键值为key的缓存是否存在</td>
		</tr>
		<tr>
			<td>get(final String key)</td>
			<td>final String key（缓存key）</td>
			<td>T extends Serializable（声明的返回类型对象）</td>
			<td>获取对应键值的缓存对象</td>
		</tr>
		<tr>
			<td>hget(final String key, final String fieldName)</td>
			<td>final String key（缓存Map的key），final String fieldName（Map下某个属性的key）</td>
			<td>T extends Serializable（声明的返回类型对象）</td>
			<td>获取HashMap中对应的子key的缓存值</td>
		</tr>
		<tr>
			<td>hmget(final String key, final String... fieldName)</td>
			<td>final String key（缓存Map的key）, final String... fieldName（Map下若干属性的key）</td>
			<td>List</td>
			<td>获取HashMap中对应的多个子key的缓存值集合</td>
		</tr>
		<tr>
			<td>hexists(final String key, final String field)</td>
			<td>final String key（缓存Map的key），final String field（Map下某个属性的key）</td>
			<td>Boolean（是否存在的标志）</td>
			<td>判断HashMap中对应的key是否存在</td>
		</tr>
		<tr>
			<td>hset(final String key, final String fieldName, final T value)</td>
			<td>final String key（缓存Map的key），final String fieldName（Map下某个属性的key），final T value（要放置的缓存对象）</td>
			<td>void</td>
			<td>放置HashMap中的属性和值</td>
		</tr>
		<tr>
			<td>removeCache(String key)</td>
			<td>String key（缓存的key）</td>
			<td>void</td>
			<td>删除key对应的缓存信息</td>
		</tr>
		<tr>
			<td>hdel(String key, String field)</td>
			<td>String key（缓存Map的key），String field（Map下某个属性的key）</td>
			<td>void</td>
			<td>删除Map下键值为filed的属性</td>
		</tr>
	</tbody>
</table>
