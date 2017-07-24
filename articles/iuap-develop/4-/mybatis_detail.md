# 使用向导
[TOC]

## 数据库建模

## 根据数据库模型生成实体类和Mybatis相关文件
本部分主要介绍基于mybatis对数据库进行操作时，需要生成的类、接口、mapperXMl文件、以及相关配置项的配置方法。

### Spring-Mybatis集成配置
iuap平台使用iuap-mybatis组件为MyBatis持久化提供支持。另外iuap-persistence同时依赖了基础的Spring Data JPA和Mybatis，如果项目上明确只使用Mybatis，则不需要引入iuap-persistence组件，直接依赖iuap-mybatis组件即可，注意要和iuap其他组件的版本保持一致。

依赖引入，在工程的pom文件中加入下面的配置

	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-mybatis</artifactId>
	  <version>3.1.0-RELEASE</version>
	</dependency>




#### java类扫描注解
iuap-mybatis组件通过@MyBatisRepository注解,确定服务接口。所以要在提供服务接口上加上该注解。如下：

	@MyBatisRepository
	public interface CardTableMapper {
	/**
		类内容
	**/
	}
		



#### xml扫描路径

mybatis使用XML文件存储在mapper接口中相应的sql方法。使用这些方法要通过配置文件去指定扫描这些XML文件的路径，同时在改配置文件中也要指定对应的实体类、是否使用分页插件等配置。

      <!-- MyBatis配置 -->
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <!-- 自动扫描entity目录, 省掉Configuration.xml里的手工配置 -->
            <property name="typeAliasesPackage"
                      value="com.yonyou.iuap.example.entity,com.yonyou.metadata.spi,com.yonyou.metadata.mybatis.util.publish.model"/>
            <!-- 显式指定Mapper文件位置 -->
            <property name="mapperLocations">
            	<!--切换数据库类型后，需要修改此处的配置文件，使用对应的数据库类型下的mapper文件-->
                <array>
                    <value>classpath*:/mybatis/pg/*.xml</value>
                    <!-- <value>classpath*:/mybatis/mysql/*.xml</value> -->
                    <!-- <value>classpath*:/mybatis/oracle/*.xml</value> -->
                    <value>classpath*:/mybatis/MetadataServiceDao/*.xml</value>
                </array>
            </property>
            <!-- 如果想用iuap的分页插件，可以放开下面注释， oracle或mysql，postgresql数据库则使用下面配置， -->
            <property name="plugins">
                <array>
                    <bean id="paginationInterceptor"
                          class="com.yonyou.iuap.mybatis.plugins.PaginationInterceptor">
                        <property name="properties">
                            <props>
                            	<!--修改数据库类型后，dbms的属性需要修改，如oracle、postgresql、mysql-->
                                <prop key="dbms">postgresql</prop>
                                <prop key="sqlRegex">.*selectAllByPage</prop>
                            </props>
                        </property>
                    </bean>
                </array>
            </property>
        </bean>
        <!-- 扫描basePackage下所有以@MyBatisRepository标识的 接口-->
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="basePackage" value="com.yonyou.iuap.example.repository,com.yonyou.metadata.mybatis.dao"/>
            <property name="annotationClass" value="com.yonyou.iuap.persistence.mybatis.anotation.MyBatisRepository"/>
            <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        </bean>


### 示例完整结构
实例中的mybatis应用，分为三个部分，1、数据库实体类，2、实体类对应的mapper接口类，3、mapper接口类对应的XML（保存sql查询代码）。

####数据库实体类
该类作为实体对应数据库中的cardtable表

	package com.yonyou.iuap.example.entity.mybatis;
	
	import java.util.Date;
	
	/**
	 * 卡片表示例实体，mybatis方式
	 */
	public class CardTable {
	    private String pksystem;
	
	    private String name;
	
	    private String code;
	
	    private String i18n;
	
	    private String gateway;
	
	    private String radiodatacontroller;
	
	    private String radiodataassociate;
	
	    private String combodatamodel;
	
	    private String tag;
	
	    private String secretkey;
	
	    private String system;
	
	    private String tenant;
	
	    private Date ts;
	
	    private Integer dr;
	
	    public String getPksystem() {
	        return pksystem;
	    }
	
	    public void setPksystem(String pksystem) {
	        this.pksystem = pksystem;
	    }
	
	    public void setTs(Date ts) {
	        this.ts = ts;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name == null ? null : name.trim();
	    }
	
	    public String getCode() {
	        return code;
	    }
	
	    public void setCode(String code) {
	        this.code = code == null ? null : code.trim();
	    }
	
	    public String getI18n() {
	        return i18n;
	    }
	
	    public void setI18n(String i18n) {
	        this.i18n = i18n == null ? null : i18n.trim();
	    }
	
	    public String getGateway() {
	        return gateway;
	    }
	
	    public void setGateway(String gateway) {
	        this.gateway = gateway == null ? null : gateway.trim();
	    }
	
	    public String getRadiodatacontroller() {
	        return radiodatacontroller;
	    }
	
	    public void setRadiodatacontroller(String radiodatacontroller) {
	        this.radiodatacontroller = radiodatacontroller == null ? null : radiodatacontroller.trim();
	    }
	
	    public String getRadiodataassociate() {
	        return radiodataassociate;
	    }
	
	    public void setRadiodataassociate(String radiodataassociate) {
	        this.radiodataassociate = radiodataassociate == null ? null : radiodataassociate.trim();
	    }
	
	    public String getCombodatamodel() {
	        return combodatamodel;
	    }
	
	    public void setCombodatamodel(String combodatamodel) {
	        this.combodatamodel = combodatamodel == null ? null : combodatamodel.trim();
	    }
	
	    public String getTag() {
	        return tag;
	    }
	
	    public void setTag(String tag) {
	        this.tag = tag == null ? null : tag.trim();
	    }
	
	    public String getSecretkey() {
	        return secretkey;
	    }
	
	    public void setSecretkey(String secretkey) {
	        this.secretkey = secretkey;
	    }
	
	    public String getSystem() {
	        return system;
	    }
	
	    public void setSystem(String system) {
	        this.system = system == null ? null : system.trim();
	    }
	
	    public String getTenant() {
	        return tenant;
	    }
	
	    public void setTenant(String tenant) {
	        this.tenant = tenant == null ? null : tenant.trim();
	    }
	
	    public Date getTs() {
	        return ts;
	    }
	
	    public Integer getDr() {
	        return dr;
	    }
	
	    public void setDr(Integer dr) {
	        this.dr = dr;
	    }
	}

####实体类对应的mapper接口类
该接口类规定了相关服务提供的方法

其中批量处理的方法	，参数传入List<实体类类型>。

	package com.yonyou.iuap.example.repository.mybatis;
	
	import com.yonyou.iuap.example.entity.mybatis.CardTable;
	import com.yonyou.iuap.mvc.type.SearchParams;
	import com.yonyou.iuap.mybatis.type.PageResult;
	import com.yonyou.iuap.persistence.mybatis.anotation.MyBatisRepository;
	
	import org.apache.ibatis.annotations.Param;
	import org.springframework.data.domain.PageRequest;
	
	import java.util.List;
	
	
	@MyBatisRepository
	public interface CardTableMapper {
		
		/**
		 * 单个增删改查
		 * */
		int insert(CardTable record);
		
		int insertSelective(CardTable record);
		
		int updateByPrimaryKeySelective(CardTable record);
	
		int updateByPrimaryKey(CardTable record);
	
		int deleteByPrimaryKey(String pksystem);
	
		CardTable selectByPrimaryKey(String pksystem);
		
		/**
		 * 根据某一非主键字段查询实体
		 */
		List<CardTable> findByCode(String code);
		
		/**
		 *分页相关
		 */
		long getCountByPage(@Param("page") PageRequest pageRequest, @Param("search") SearchParams searchParams);
	    
		PageResult<CardTable> selectAllByPage(@Param("page") PageRequest pageRequest, @Param("search") SearchParams searchParams);
	    
		/**
		 *批量操作
		 */
	    void batchInsert(List<CardTable> addList);
	
	    void batchUpdate(List<CardTable> updateList);
	
	    void batchDeleteByPrimaryKey(List<CardTable> list);
	    
	}

####与mapper接口类对应的XML文件
在该文件中resultMap是实体类CardTable的映射，请保证resultMap标签的type属性指向对应的实体类， 不同的字段对应了不同的jdbcType。
    
    <insert>、<select>、<update>、<delete>标签中定义的方法名要与接口类方法名相同，其中批量操作的方法，通过<foreach>标签遍历传入参数进行处理。


	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
	<mapper namespace="com.yonyou.iuap.example.repository.mybatis.CardTableMapper">
		<resultMap id="BaseResultMap" type="com.yonyou.iuap.example.entity.mybatis.CardTable">
			<id column="pk_system" property="pksystem" jdbcType="VARCHAR" />
			<result column="name" property="name" jdbcType="VARCHAR" />
			<result column="code" property="code" jdbcType="VARCHAR" />
			<result column="i18n" property="i18n" jdbcType="VARCHAR" />
			<result column="gateway" property="gateway" jdbcType="VARCHAR" />
			<result column="radiodatacontroller" property="radiodatacontroller"
				jdbcType="VARCHAR" />
			<result column="radiodataassociate" property="radiodataassociate"
				jdbcType="VARCHAR" />
			<result column="combodatamodel" property="combodatamodel"
				jdbcType="VARCHAR" />
			<result column="tag" property="tag" jdbcType="VARCHAR" />
			<result column="secretkey" property="secretkey" jdbcType="VARCHAR" />
			<result column="system" property="system" jdbcType="VARCHAR" />
			<result column="tenant" property="tenant" jdbcType="VARCHAR" />
			<result column="ts" property="ts" jdbcType="TIMESTAMP" />
			<result column="dr" property="dr" jdbcType="NUMERIC" />
		</resultMap>
		<sql id="Base_Column_List">
			pk_system, name, code, i18n, gateway, radiodatacontroller, radiodataassociate,
			combodatamodel,
			tag,secretkey, system, tenant, ts, dr
		</sql>
		<select id="selectByPrimaryKey" resultMap="BaseResultMap"
			parameterType="java.lang.String">
			select
			<include refid="Base_Column_List" />
			from cardtable
			where pk_system = #{pksystem,jdbcType=VARCHAR}
		</select>
	
		<select id="findByCode" resultMap="BaseResultMap"
			parameterType="java.lang.String">
			select
			<include refid="Base_Column_List" />
			from cardtable
			where code = #{code}
		</select>
	
	
		<select id="selectAllByPage" resultMap="BaseResultMap"
			resultType="java.util.List">
			SELECT
			<include refid="Base_Column_List" />
			from cardtable where 1=1
			<if test="search != null">
				<if
					test="search.searchMap.searchParam!=null and search.searchMap.searchParam!='' ">
					and code like CONCAT(CONCAT('%', #{search.searchMap.searchParam}),'%') or name like CONCAT(CONCAT('%', #{search.searchMap.searchParam}),'%')
				</if>
			</if>
		</select>
	
		<delete id="deleteByPrimaryKey" parameterType="java.lang.String">
			delete from cardtable
			where pk_system = #{pksystem,jdbcType=VARCHAR}
		</delete>
	
		<delete id="batchDeleteByPrimaryKey" parameterType="java.util.List">
			delete from cardtable
			where
			pk_system in
			<foreach collection="list" item="item" index="index"
				separator="," open="(" close=")">
			#{item.pksystem,jdbcType=VARCHAR}
			</foreach>
		</delete>
	
		<insert id="insert" parameterType="com.yonyou.iuap.example.entity.mybatis.CardTable">
			insert into cardtable (pk_system, name, code,
			i18n, gateway, radiodatacontroller,
			radiodataassociate, combodatamodel, tag,
			secretkey,system, tenant, ts, dr
			)
			values (#{pksystem,jdbcType=VARCHAR}, #{name,jdbcType=VARCHAR},
			#{code,jdbcType=VARCHAR},
			#{i18n,jdbcType=VARCHAR}, #{gateway,jdbcType=VARCHAR}, #{radiodatacontroller,jdbcType=VARCHAR},
			#{radiodataassociate,jdbcType=VARCHAR},
			#{combodatamodel,jdbcType=VARCHAR}, #{tag,jdbcType=VARCHAR},
			#{secretkey,jdbcType=VARCHAR},#{system,jdbcType=VARCHAR},
			#{tenant,jdbcType=VARCHAR}, #{ts,jdbcType=TIMESTAMP},
			#{dr,jdbcType=NUMERIC}
			)
		</insert>
	
		<insert id="batchInsert" parameterType="java.util.List">
			insert into cardtable (pk_system, name, code,
			i18n, gateway, radiodatacontroller,
			radiodataassociate, combodatamodel, tag,
			secretkey,system, tenant, ts, dr
			)
			values
			<foreach collection="list" item="item" index="index"
				separator=",">
				(#{item.pksystem,jdbcType=VARCHAR}, #{item.name,jdbcType=VARCHAR}, #{item.code,jdbcType=VARCHAR},
				#{item.i18n,jdbcType=VARCHAR}, #{item.gateway,jdbcType=VARCHAR},
				#{item.radiodatacontroller,jdbcType=VARCHAR},
				#{item.radiodataassociate,jdbcType=VARCHAR},
				#{item.combodatamodel,jdbcType=VARCHAR},
				#{item.tag,jdbcType=VARCHAR},
				#{item.secretkey,jdbcType=VARCHAR},#{item.system,jdbcType=VARCHAR},
				#{item.tenant,jdbcType=VARCHAR}, #{item.ts,jdbcType=TIMESTAMP},
				#{item.dr,jdbcType=NUMERIC}
				)
			</foreach>
		</insert>
	
		<insert id="insertSelective" parameterType="com.yonyou.iuap.example.entity.mybatis.CardTable">
			insert into cardtable
			<trim prefix="(" suffix=")" suffixOverrides=",">
				<if test="pksystem != null">
					pk_system,
				</if>
				<if test="name != null">
					name,
				</if>
				<if test="code != null">
					code,
				</if>
				<if test="i18n != null">
					i18n,
				</if>
				<if test="gateway != null">
					gateway,
				</if>
				<if test="radiodatacontroller != null">
					radiodatacontroller,
				</if>
				<if test="radiodataassociate != null">
					radiodataassociate,
				</if>
				<if test="combodatamodel != null">
					combodatamodel,
				</if>
				<if test="tag != null">
					tag,
				</if>
				<if test="secretkey != null">
					secretkey,
				</if>
				<if test="system != null">
					system,
				</if>
				<if test="tenant != null">
					tenant,
				</if>
				<if test="ts != null">
					ts,
				</if>
				<if test="dr != null">
					dr,
				</if>
			</trim>
			<trim prefix="values (" suffix=")" suffixOverrides=",">
				<if test="pksystem != null">
					#{pksystem,jdbcType=VARCHAR},
				</if>
				<if test="name != null">
					#{name,jdbcType=VARCHAR},
				</if>
				<if test="code != null">
					#{code,jdbcType=VARCHAR},
				</if>
				<if test="i18n != null">
					#{i18n,jdbcType=VARCHAR},
				</if>
				<if test="gateway != null">
					#{gateway,jdbcType=VARCHAR},
				</if>
				<if test="radiodatacontroller != null">
					#{radiodatacontroller,jdbcType=VARCHAR},
				</if>
				<if test="radiodataassociate != null">
					#{radiodataassociate,jdbcType=VARCHAR},
				</if>
				<if test="combodatamodel != null">
					#{combodatamodel,jdbcType=VARCHAR},
				</if>
				<if test="tag != null">
					#{tag,jdbcType=VARCHAR},
				</if>
				<if test="secretkey != null">
					#{secretkey,jdbcType=VARCHAR},
				</if>
				<if test="system != null">
					#{system,jdbcType=VARCHAR},
				</if>
				<if test="tenant != null">
					#{tenant,jdbcType=VARCHAR},
				</if>
				<if test="ts != null">
					#{ts,jdbcType=TIMESTAMP},
				</if>
				<if test="dr != null">
					#{dr,jdbcType=NUMERIC},
				</if>
			</trim>
		</insert>
		<update id="updateByPrimaryKeySelective" parameterType="com.yonyou.iuap.example.entity.mybatis.CardTable">
			update cardtable
			<set>
				<if test="name != null">
					name = #{name,jdbcType=VARCHAR},
				</if>
				<if test="code != null">
					code = #{code,jdbcType=VARCHAR},
				</if>
				<if test="i18n != null">
					i18n = #{i18n,jdbcType=VARCHAR},
				</if>
				<if test="gateway != null">
					gateway = #{gateway,jdbcType=VARCHAR},
				</if>
				<if test="radiodatacontroller != null">
					radiodatacontroller = #{radiodatacontroller,jdbcType=VARCHAR},
				</if>
				<if test="radiodataassociate != null">
					radiodataassociate = #{radiodataassociate,jdbcType=VARCHAR},
				</if>
				<if test="combodatamodel != null">
					combodatamodel = #{combodatamodel,jdbcType=VARCHAR},
				</if>
				<if test="tag != null">
					tag = #{tag,jdbcType=VARCHAR},
				</if>
				<if test="secretkey != null">
					secretkey = #{secretkey,jdbcType=VARCHAR},
				</if>
				<if test="system != null">
					system = #{system,jdbcType=VARCHAR},
				</if>
				<if test="tenant != null">
					tenant = #{tenant,jdbcType=VARCHAR},
				</if>
				<if test="dr != null">
					dr = #{dr,jdbcType=NUMERIC},
				</if>
			</set>
			where pk_system = #{pksystem,jdbcType=VARCHAR}
		</update>
		<update id="updateByPrimaryKey" parameterType="com.yonyou.iuap.example.entity.mybatis.CardTable">
			update cardtable
			set name = #{name,jdbcType=VARCHAR},
			code = #{code,jdbcType=VARCHAR},
			i18n = #{i18n,jdbcType=VARCHAR},
			gateway = #{gateway,jdbcType=VARCHAR},
			radiodatacontroller = #{radiodatacontroller,jdbcType=VARCHAR},
			radiodataassociate = #{radiodataassociate,jdbcType=VARCHAR},
			combodatamodel = #{combodatamodel,jdbcType=VARCHAR},
			tag = #{tag,jdbcType=VARCHAR},
			secretkey = #{secretkey,jdbcType=VARCHAR},
			system = #{system,jdbcType=VARCHAR},
			tenant = #{tenant,jdbcType=VARCHAR},
			dr = #{dr,jdbcType=NUMERIC}
			where pk_system = #{pksystem,jdbcType=VARCHAR}
		</update>
	
		<update id="batchUpdate" parameterType="java.util.List">
			<foreach collection="list" item="item" index="index" open=""
				close="" separator=";">
				update cardtable
				set name = #{item.name,jdbcType=VARCHAR},
				code = #{item.code,jdbcType=VARCHAR},
				i18n = #{item.i18n,jdbcType=VARCHAR},
				gateway = #{item.gateway,jdbcType=VARCHAR},
				radiodatacontroller = #{item.radiodatacontroller,jdbcType=VARCHAR},
				radiodataassociate = #{item.radiodataassociate,jdbcType=VARCHAR},
				combodatamodel = #{item.combodatamodel,jdbcType=VARCHAR},
				tag = #{item.tag,jdbcType=VARCHAR},
				secretkey = #{item.secretkey,jdbcType=VARCHAR},
				system = #{item.system,jdbcType=VARCHAR},
				tenant = #{item.tenant,jdbcType=VARCHAR},
				dr = #{item.dr,jdbcType=NUMERIC}
				where pk_system = #{item.pksystem,jdbcType=VARCHAR}
			</foreach>
		</update>
	</mapper>

### Mapper.xml编写规则

#### 命名空间
在mapper xml文件中

&lt;mapper namespace="com.yonyou.iuap.example.repository.mybatis.CardTableMapper"&gt;

这行配置设定了该xml文件的命名空间，该命名空间指向了与xml文件相匹配的mapper接口类，通过设定命名空间你可以不用写接口实现类，mybatis会通过该绑定自动帮你找到对应要执行的SQL语句。

#### ResultMap
ResultMap配置内容

	<resultMap id="BaseResultMap" type="com.yonyou.iuap.example.entity.mybatis.CardTable">
		<id column="pk_system" property="pksystem" jdbcType="VARCHAR" />
		<result column="name" property="name" jdbcType="VARCHAR" />
		<result column="code" property="code" jdbcType="VARCHAR" />
		<result column="i18n" property="i18n" jdbcType="VARCHAR" />
		<result column="gateway" property="gateway" jdbcType="VARCHAR" />
		<result column="radiodatacontroller" property="radiodatacontroller"
			jdbcType="VARCHAR" />
		<result column="radiodataassociate" property="radiodataassociate"
			jdbcType="VARCHAR" />
		<result column="combodatamodel" property="combodatamodel"
			jdbcType="VARCHAR" />
		<result column="tag" property="tag" jdbcType="VARCHAR" />
		<result column="secretkey" property="secretkey" jdbcType="VARCHAR" />
		<result column="system" property="system" jdbcType="VARCHAR" />
		<result column="tenant" property="tenant" jdbcType="VARCHAR" />
		<result column="ts" property="ts" jdbcType="TIMESTAMP" />
		<result column="dr" property="dr" jdbcType="NUMERIC" />
	</resultMap>

该配置制定了sql语句返回结果对应项目中哪一个实体类，它包含以下几个标签：

id属性 ：resultMap标签的标识。

type属性：Java实体类的类名

result column标签：数据库表对应的各个列

javaType属性 ：实体类成员字段的java类型

jdbcType属性 ：实体类成员字段的jdbc类型


#### Mapper类方法名和xml对应关系
xml文件命名空间指定的接口类所包含的方法，要与xml文件中实现的方法一一对应。

#### 各个数据库对于save，update，remove的批量处理
**增加**

增加操作三种数据库基本没有区别。

postgresql

	<insert id="batchInsert" parameterType="java.util.List">
		insert into cardtable (pk_system, name, code,
		i18n, gateway, radiodatacontroller,
		radiodataassociate, combodatamodel, tag,
		secretkey,system, tenant, ts, dr
		)
		values
		<foreach collection="list" item="item" index="index"
			separator=",">
			(#{item.pksystem,jdbcType=VARCHAR}, #{item.name,jdbcType=VARCHAR}, #{item.code,jdbcType=VARCHAR},
			#{item.i18n,jdbcType=VARCHAR}, #{item.gateway,jdbcType=VARCHAR},
			#{item.radiodatacontroller,jdbcType=VARCHAR},
			#{item.radiodataassociate,jdbcType=VARCHAR},
			#{item.combodatamodel,jdbcType=VARCHAR},
			#{item.tag,jdbcType=VARCHAR},
			#{item.secretkey,jdbcType=VARCHAR},#{item.system,jdbcType=VARCHAR},
			#{item.tenant,jdbcType=VARCHAR}, #{item.ts,jdbcType=TIMESTAMP},
			#{item.dr,jdbcType=NUMERIC}
			)
		</foreach>
	</insert>

mysql

	<insert id="batchInsert" parameterType="java.util.List">
		insert into cardtable (pk_system, name, code,
		i18n, gateway, radiodatacontroller,
		radiodataassociate, combodatamodel, tag,
		secretkey,system, tenant, ts, dr
		)
		values
		<foreach collection="list" item="item" index="index"
			separator=",">
			(#{item.pksystem,jdbcType=VARCHAR}, #{item.name,jdbcType=VARCHAR}, #{item.code,jdbcType=VARCHAR},
			#{item.i18n,jdbcType=VARCHAR}, #{item.gateway,jdbcType=VARCHAR},
			#{item.radiodatacontroller,jdbcType=VARCHAR},
			#{item.radiodataassociate,jdbcType=VARCHAR},
			#{item.combodatamodel,jdbcType=VARCHAR},
			#{item.tag,jdbcType=VARCHAR},
			#{item.secretkey,jdbcType=VARCHAR},#{item.system,jdbcType=VARCHAR},
			#{item.tenant,jdbcType=VARCHAR}, #{item.ts,jdbcType=TIMESTAMP},
			#{item.dr,jdbcType=NUMERIC}
			)
		</foreach>
	</insert>

oracle

	<insert id="batchInsert" parameterType="java.util.List">
		insert into cardtable (pk_system, name, code,
		i18n, gateway, radiodatacontroller,
		radiodataassociate, combodatamodel, tag,
		secretkey,system, tenant, ts, dr
		)
		values
		<foreach collection="list" item="item" index="index"
			separator=",">
			(#{item.pksystem,jdbcType=VARCHAR}, #{item.name,jdbcType=VARCHAR}, #{item.code,jdbcType=VARCHAR},
			#{item.i18n,jdbcType=VARCHAR}, #{item.gateway,jdbcType=VARCHAR},
			#{item.radiodatacontroller,jdbcType=VARCHAR},
			#{item.radiodataassociate,jdbcType=VARCHAR},
			#{item.combodatamodel,jdbcType=VARCHAR},
			#{item.tag,jdbcType=VARCHAR},
			#{item.secretkey,jdbcType=VARCHAR},#{item.system,jdbcType=VARCHAR},
			#{item.tenant,jdbcType=VARCHAR}, #{item.ts,jdbcType=TIMESTAMP},
			#{item.dr,jdbcType=NUMERIC}
			)
		</foreach>
	</insert>

**修改**

批量修改操作中pg和mysql数据库没有区别，oracle数据库需要加上begin和end标志

postgresql

	<update id="batchUpdate" parameterType="java.util.List">
		<foreach collection="list" item="item" index="index" open=""
			close="" separator=";">
			update cardtable
			set name = #{item.name,jdbcType=VARCHAR},
			code = #{item.code,jdbcType=VARCHAR},
			i18n = #{item.i18n,jdbcType=VARCHAR},
			gateway = #{item.gateway,jdbcType=VARCHAR},
			radiodatacontroller = #{item.radiodatacontroller,jdbcType=VARCHAR},
			radiodataassociate = #{item.radiodataassociate,jdbcType=VARCHAR},
			combodatamodel = #{item.combodatamodel,jdbcType=VARCHAR},
			tag = #{item.tag,jdbcType=VARCHAR},
			secretkey = #{item.secretkey,jdbcType=VARCHAR},
			system = #{item.system,jdbcType=VARCHAR},
			tenant = #{item.tenant,jdbcType=VARCHAR},
			dr = #{item.dr,jdbcType=NUMERIC}
			where pk_system = #{item.pksystem,jdbcType=VARCHAR}
		</foreach>
	</update>


mysql

	<update id="batchUpdate" parameterType="java.util.List">
		<foreach collection="list" item="item" index="index" open=""
			close="" separator=";">
			update cardtable
			set name = #{item.name,jdbcType=VARCHAR},
			code = #{item.code,jdbcType=VARCHAR},
			i18n = #{item.i18n,jdbcType=VARCHAR},
			gateway = #{item.gateway,jdbcType=VARCHAR},
			radiodatacontroller = #{item.radiodatacontroller,jdbcType=VARCHAR},
			radiodataassociate = #{item.radiodataassociate,jdbcType=VARCHAR},
			combodatamodel = #{item.combodatamodel,jdbcType=VARCHAR},
			tag = #{item.tag,jdbcType=VARCHAR},
			secretkey = #{item.secretkey,jdbcType=VARCHAR},
			system = #{item.system,jdbcType=VARCHAR},
			tenant = #{item.tenant,jdbcType=VARCHAR},
			dr = #{item.dr,jdbcType=NUMERIC}
			where pk_system = #{item.pksystem,jdbcType=VARCHAR}
		</foreach>
	</update>

oracle

	<update id="batchUpdate" parameterType="java.util.List">
	 begin 
		<foreach collection="list" item="item" index="index" open="" close="" separator=";">
			update cardtable
			<set>
				name = #{item.name,jdbcType=VARCHAR},
				code = #{item.code,jdbcType=VARCHAR},
				i18n = #{item.i18n,jdbcType=VARCHAR},
				gateway = #{item.gateway,jdbcType=VARCHAR},
				radiodatacontroller = #{item.radiodatacontroller,jdbcType=VARCHAR},
				radiodataassociate = #{item.radiodataassociate,jdbcType=VARCHAR},
				combodatamodel = #{item.combodatamodel,jdbcType=VARCHAR},
				tag = #{item.tag,jdbcType=VARCHAR},
				secretkey = #{item.secretkey,jdbcType=VARCHAR},
				system = #{item.system,jdbcType=VARCHAR},
				tenant = #{item.tenant,jdbcType=VARCHAR},
				dr = #{item.dr,jdbcType=NUMERIC}
			</set>
			where pk_system = #{item.pksystem,jdbcType=VARCHAR}
		</foreach>
		;end;
	</update>

**删除**

postgresql

	<delete id="batchDeleteByPrimaryKey" parameterType="java.util.List">
		delete from cardtable
		where
		pk_system in
		<foreach collection="list" item="item" index="index"
			separator="," open="(" close=")">
		#{item.pksystem,jdbcType=VARCHAR}
		</foreach>
	</delete>

mysql

	<delete id="batchDeleteByPrimaryKey" parameterType="java.util.List">
		delete from cardtable
		where
		pk_system in
		<foreach collection="list" item="item" index="index"
			separator="," open="(" close=")">
		#{item.pksystem,jdbcType=VARCHAR}
		</foreach>
	</delete>

oracle

	<delete id="batchDeleteByPrimaryKey" parameterType="java.util.List">
		delete from cardtable
		where
		pk_system in
		<foreach collection="list" item="item" index="index"
			separator="," open="(" close=")">
		#{item.pksystem,jdbcType=VARCHAR}
		</foreach>
	</delete>


## Service层编写规则以及事务处理

以样例中的 档案功能为例说明

### 包扫描规则的配置
   在 applicationContext.xml 配置文件中加入包路径扫描
   
         <!-- 使用annotation 自动注册bean, 并保证@Required、@Autowired的属性被注入 -->
	<context:component-scan base-package="com.yonyou.iuap,uap.iweb,com.yonyou.metadata.mybatis.service,com.yonyou.metadata.caches.redis">
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
		<context:exclude-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
	</context:component-scan>


多个包名，用英文逗号隔开（,） 。


### 事务注解使用说明（简略）
java代码的service层，java目录结构类似 ： com.yonyou.iuap.example.service ,
对保存、修改、删除操作进行事务控制，用spring事务注解来实现。
service通过 @Autowired注解引入 dao层

    import org.springframework.transaction.annotation.Transactional;
    
     @Autowired
     private SysDictTypeMapper sysDictTypeMapper;

    @Transactional
    public void batchDeleteByPrimaryKey(List<SysDictType> list) {
        if (CollectionUtils.isEmpty(list)) {
            throw new WebRuntimeException("当前没有选中数据!");
        }
        sysDictTypeMapper.batchDeleteByPrimaryKey(list);
    }


### 业务校验规则
业务中需要进行校验的，如编码不能重复。对校验不通过的直接抛出校验异常。
代码如（参见示例中 com.yonyou.iuap.example.validator.SysDictTypeValidator.java）

    public void valid(List<SysDictType> sysDictTypeList) {
        StringBuilder builder = new StringBuilder();
        for (SysDictType dictType : sysDictTypeList) {
            if (sysDictTypeService.findByCode(dictType.getDicttypecode()) != null) {
                builder.append(dictType.getDicttypecode()).append(",");
            }
        }
        if (builder.toString().length() > 0) {
            String codeStr = builder.deleteCharAt(builder.length() - 1).toString();
            StringBuilder msgBuilder = new StringBuilder("编码为");
            msgBuilder.append(codeStr).append("的记录已经存在!");
            throw new ValidException(msgBuilder.toString());
        }
    }

service常用方法:

- public Page<SysDictType> selectAllByPage(PageRequest pageRequest, Map<String, Object> searchParams)；

- public void save(List<SysDictType> addList, List<SysDictType> updateList, List<SysDictType> removeList)

- public void batchDeleteByPrimaryKey(List<SysDictType> list) ；

- public SysDictType findByCode(String dictTypeCode) ；

方法名|说明| 参数 | 返回值
---|---|---|---
selectAllByPage | 根据条件进行分页查询|参数1：PageRequest，分页信息，参数2：Map<String, Object> 查询参数        map的key值为对应的bean的属性值，value为查询的值|Page<T>
save | 保存数据 |参数1：新添加数据集合，参数2：更新数据集合，参数3：删除数据集合|
batchDeleteByPrimaryKey|批量删除|参数1：要删除数据集合|
findByCode|根据编码查询，可用于校验编码重复问题|参数1：编码值 |




### 异常处理(throw Exception类型说明)
对异常进行统一处理， 在spring-mvc.xml配置异常处理类

    <bean class="com.yonyou.iuap.mvc.exceptionhandle.CustomExceptionResolver"/>

配置完成后，在编码过程中就可以直接将异常抛出，前端就可以显示抛出的异常信息，进行提示。
写法可见上面所抛出的 代码校验异常。

      throw new ValidException('编码已经存在');
      

校验异常中分为全局异常，和 局部异常，全局异常在页面上统一进行提示， 局部异常可以针对 每一个字段进行提示。
如果程序中不需要对每一个字段进行提示，则抛出全局异常即可。（示例中抛出的为全局异常）
      
还可以抛出 其他运行时异常 如：

      /**
       * 批量删除
      * @param list
      */
     @Transactional
    public void batchDeleteByPrimaryKey(List<SysDictType> list) {
        if (CollectionUtils.isEmpty(list)) {
            throw new WebRuntimeException("当前没有选中数据!");
        }
        sysDictTypeMapper.batchDeleteByPrimaryKey(list);
    }




### http请求处理以及参数转换
iuap针对通用的分页参数和查询参数做了类型转换，对分页参数转换为`Pagerequest`，查询参数转换为`SearchParams`。

#### spring 配置
需要在`spring-mvc.xml`中添加参数转换类`com.yonyou.iuap.mvc.RequestArgumentResolver`。
	
	<mvc:annotation-driven>
        <mvc:message-converters register-defaults="true">
            <!-- 将StringHttpMessageConverter的默认编码设为UTF-8 -->
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8"/>
            </bean>
            <!-- 将Jackson2HttpMessageConverter的默认格式化输出设为true -->
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="prettyPrint" value="false"/>
            </bean>
        </mvc:message-converters>
        <mvc:argument-resolvers>
            <bean class="com.yonyou.iuap.mvc.RequestArgumentResolver"/>
        </mvc:argument-resolvers>
    </mvc:annotation-driven>

#### 使用说明
- 请求url

	http://localhost:8080/iuap-quickstart/sysUser/list?pageIndex=0&pageSize=5&sortField=create_time&sortDirection=desc&search_name=test&search_code=0x0004

- Controller的方法

	public JsonResponse list(PageRequest pageRequest, @FrontModelExchange(modelType = CardTable.class) SearchParams searchParams)

- 转换参数说明
<table>
	<tr><th>参数名</th><th>说明</th></tr>
	<tr><td>pageIndex</td><td>当前页数</td></tr>
	<tr><td>pageSize</td><td>每页显示的数据条数</td></tr>
	<tr><td>sortField</td><td>排序的列名称</td></tr>
	<tr><td>sortDirection</td><td>排序规则(asc|desc)</td></tr>
	<tr><td>search_*</td><td>查询条件，以search开头，后面是实体类的属性名</td></tr>
</table>

- 注解`@FrontModelExchange`

  如果不加`@FrontModelExchange`，`SearchParams`中所有的参数都是`String`类型；添加注解之后，参数会转换为实体Bean中的参数类型。

#### 注意事项

-  必须是`Get`请求

### http响应数据结构
统一使用`JsonReponse`作为`Controller`返回的数据结构,规范数据模型。
具体参考`BaseController`中的`buildSuccess()`和`buildError()`。

### 异常处理
#### spring-mvc中配置统一异常处理的handler

    <bean class="com.yonyou.iuap.mvc.exceptionhandle.CustomExceptionResolver"/>
#### 异常返回的数据结构
统一使用`JsonErrorReponse`作为返回的数据结构。

#### 异常处理方式

对于ajax请求或者http header中包含Accept:application/json 的请求，会返回json数据；
对于其他请求，会跳转到springmvc视图下的error/500页面。

### 完整的Controller配置及代码
- Controller代码
	
		@RestController
		@RequestMapping(value = "/cardtable")
		public class CardTableController extends BaseController {
		    @Autowired
		    private CardTableService cardTableService;
		
		    /**
		     * 查询分页数据
		     * 
		     * @param pageRequest
		     * @param searchParams
		     * @return
		     */
		    @RequestMapping(value = "/list", method = RequestMethod.GET)
		    public @ResponseBody JsonResponse page(PageRequest pageRequest, SearchParams searchParams) {
		        Page<CardTable> data = cardTableService.selectAllByPage(pageRequest, searchParams);
		        return buildSuccess(data);
		    }
		
		    /**
		     * 保存数据
		     * 
		     * @param list
		     * @return
		     */
		    @RequestMapping(value = "/save", method = RequestMethod.POST)
		    public @ResponseBody JsonResponse save(@RequestBody List<CardTable> list) {
		        cardTableService.save(list);
		        return buildSuccess();
		    }
		
		
		    /**
		     * 删除数据
		     * 
		     * @param list
		     * @return
		     */
		    @RequestMapping(value = "/del", method = RequestMethod.POST)
		    public @ResponseBody JsonResponse del(@RequestBody List<CardTable> list) {
		        cardTableService.batchDeleteByPrimaryKey(list);
		        return buildSuccess();
		    }
		}

- spring-mvc配置
	
		<!-- 自动扫描且只扫描@Controller -->
	    <context:component-scan base-package="com.yonyou.iuap" use-default-filters="false">
	        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	        <context:include-filter type="annotation"
	                                		expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
	    </context:component-scan>
	
	    <mvc:annotation-driven>
	        <mvc:message-converters register-defaults="true">
	            <!-- 将StringHttpMessageConverter的默认编码设为UTF-8 -->
	            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
	                <constructor-arg value="UTF-8"/>
	            </bean>
	            <!-- 将Jackson2HttpMessageConverter的默认格式化输出设为true -->
	            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
	                <property name="prettyPrint" value="false"/>
	            </bean>
	        </mvc:message-converters>
	        <mvc:argument-resolvers>
	            <bean class="com.yonyou.iuap.mvc.RequestArgumentResolver"/>
	        </mvc:argument-resolvers>
	    </mvc:annotation-driven>
	
	    <!-- freemarker的配置 -->
	    <bean id="freemarkerConfigurer"
	          class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
	        <property name="templateLoaderPath" value="/WEB-INF/views/"/>
	        <property name="defaultEncoding" value="UTF-8"/>
	        <property name="freemarkerSettings">
	            <props>
	                <prop key="template_update_delay">10</prop>
	                <prop key="locale">zh_CN</prop>
	                <prop key="datetime_format">yyyy-MM-dd HH:mm:ss</prop>
	                <prop key="date_format">yyyy-MM-dd</prop>
	            </props>
	        </property>
	        <property name="freemarkerVariables">
	            <map>
	                <entry key="ctx" value="/iuap-quickstart"/>
	            </map>
	        </property>
	    </bean>
	
	    <!-- FreeMarker视图解析器 -->
	    <bean id="viewResolver"
	         class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
	        <property name="viewClass" value="org.springframework.web.servlet.view.freemarker.FreeMarkerView"/>
	        <property name="suffix" value=".ftl"/>
	        <property name="contentType" value="text/html;charset=UTF-8"/>
	        <property name="exposeRequestAttributes" value="true"/>
	        <property name="exposeSessionAttributes" value="true"/>
	        <property name="exposeSpringMacroHelpers" value="true"/>
	        <property name="requestContextAttribute" value="request"/>
	    </bean>
	
	    <!-- 定义JSP文件的位置 -->
	    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	        <property name="prefix" value="/WEB-INF/views/"/>
	        <property name="suffix" value=".jsp"/>
	    </bean>
	
	    <!-- 容器默认的DefaultServletHandler处理 所有静态内容与无RequestMapping处理的URL-->
	    <mvc:default-servlet-handler/>
	
	    <bean class="com.yonyou.iuap.mvc.exceptionhandle.CustomExceptionResolver"/>
	

##前端js以及html编写规则
### js
#### 前端js模型定义（meta.js）

	var metaCardTable={
					meta: {
						"name":{
							type:'string',
							required:true,
							notipFlag: true,
							hasSuccess: true,
							nullMsg:'系统名称不能为空!'
						},
						"code":{
							type:'string',
							required:true,
							notipFlag: true,
							hasSuccess: true,
							nullMsg:'系统编码不能为空!'
						},
						"radiodatacontroller":"",
						"radiodataassociate":"",
						"combodatamodel":"",
						"gateway": "",
						"pksystem":"",
						"ts":"",
						"tag": ""
					}
				};

前端js数据模型定义格式：

	var 数据模型名称={
					meta: {
						//数据模型对应的实体类字段
					}
				};

数据模型中的字段如果与后台实体类字段不一致，需要在类类名上加@JsonIgnoreProperties(ignoreUnknown = true)注解。

如果有数据模型有校验需求，可以参照如下配置：

		            worktel: {
		                type: 'string',
		                regExp: /^1[34578]\d{9}$/,
				        notipFlag: true,
				        hasSuccess: true,
				        errorMsg: "手机号码格式不对。"	
		            },	

regExp 为校验的匹配的正则表达式，errorMsg为校验不成功的错误信息。

#### 前端事件定义

前端事件都定义在了cardtable.js的viewModel.event中，其中包括了对前端增删改查按钮的响应、对各种ajax请求的发送和处理等。

				event: {					
	                //清除datatable数据
	                clearDt: function (dt) {
	                	dt.removeAllRows();
	                	dt.clear();
	                },
					// 卡片表数据读取
					initCardTableList:function(){
						var jsonData={
								pageIndex:viewModel.draw-1,
								pageSize:viewModel.pageSize,
								sortField:"createtime",
								sortDirection:"asc"
						};
						$(element).find("#search").each(function(){
							if(this.value == undefined || this.value =='' || this.value.length ==0 ){
								//不执行操作
							}else{
								jsonData['search_searchParam'] =  this.value.replace(/(^\s*)|(\s*$)/g, "");  //去掉空格
							}
						});
						$.ajax({
							type:'get',
							url:listUrl,
							datatype:'json',
							data:jsonData,
							contentType: 'application/json;charset=utf-8',
							success:function(res){
								if(res){
									if( res.success =='success'){
										if(res.detailMsg.data){
											viewModel.totleCount=res.detailMsg.data.totalElements;
											viewModel.totlePage=res.detailMsg.data.totalPages;
											viewModel.event.comps.update({totalPages:viewModel.totlePage,pageSize:viewModel.pageSize,currentPage:viewModel.draw,totalCount:viewModel.totleCount});
											viewModel.dt1.removeAllRows();
											viewModel.dt1.clear();
											viewModel.dt1.setSimpleData(res.detailMsg.data.content,{unSelect:true});
										}
									}else{
										var msg = "";
										if(res.success == 'fail_global'){
											msg = res.message;
										}else{
											for (var key in res.detailMsg) {
												msg += res.detailMsg[key] + "<br/>";
											}
										}
										u.messageDialog({msg:msg,title:'请求错误',btnText:'确定'});
									}
								}else{
									u.messageDialog({msg:'后台返回数据格式有误，请联系管理员',title:'数据错误',btnText:'确定'});
								}
							},
							error:function(er){
								u.messageDialog({msg:'请求超时',title:'请求错误',btnText:'确定'});
							}
						});
					},
					//组装list
					genDataList:function(data){
						var datalist = [];
						datalist.push(data);
						return datalist ;
					},
					//删除方法
					deleteData: function(data) {
						var datalist = viewModel.event.genDataList(data);
						var json = JSON.stringify(datalist);
						$.ajax({
							url: delUrl,
							data: json,
							dataType: 'json',
							type: 'post',
							contentType: 'application/json',
							success: function (res) {
								//md.close();
								if(res){
									if (res.success == 'success') {
										u.messageDialog({msg:'删除成功',title:'操作提示',btnText:'确定'});
									} else {
										var msg = "";
										for(var key in res.message){
											msg +=res.message[key]+"<br/>";
										}
										u.messageDialog({msg:'msg',title:'操作提示',btnText:'确定'});
									}
								}else{
									u.messageDialog({msg:'无返回数据',title:'操作提示',btnText:'确定'});
								}
								
							},
							error:function(er){
								u.messageDialog({msg:'操作请求失败，'+er,title:'操作提示',btnText:'确定'});
							}
						});
					},
					//新增和更新方法
					saveData:function(data) {
						var datalist = viewModel.event.genDataList(data);
						var json = JSON.stringify(datalist);
						$.ajax({
							url: saveUrl,
							type: 'post',
							data: json,
							dataType: 'json',
							contentType: 'application/json',
							success: function (res) {
								if(res){
									if( res.success =='success'){
			                            viewModel.event.initCardTableList();
										u.messageDialog({msg:'操作成功',title:'操作提示',btnText:'确定'});
										md.close();
									}else{
										var msg = "";
										if(res.success == 'fail_global'){
											msg = res.message;
										}else{
											for (var key in res.detailMsg) {
												msg += res.detailMsg[key] + "<br/>";
											}
										}
										u.messageDialog({msg:msg,title:'操作提示',btnText:'确定'});
									}
								}else{
									u.messageDialog({msg:'没有返回数据',title:'操作提示',btnText:'确定'});
								}
							}
						});
					},
					//分页相关
					pageChange:function(){
						viewModel.event.comps.on('pageChange', function (pageIndex) {
							viewModel.draw = pageIndex + 1;
							viewModel.event.initCardTableList();
						});
					},
					sizeChange:function(){
						viewModel.event.comps.on('sizeChange', function (arg) {
							viewModel.pageSize = parseInt(arg);
							viewModel.draw = 1;
							viewModel.event.initCardTableList();
						});
					},
					//页面初始化
					pageInit: function () {		       
						$(element).html(html) ;
						viewModel.app = u.createApp({
							el: element,
							model: viewModel
						});
						
						var paginationDiv = $(element).find('#pagination')[0];
						this.comps=new u.pagination({el:paginationDiv,jumppage:true});
						this.initCardTableList();
						viewModel.event.pageChange();
						viewModel.event.sizeChange();
						
	                    //回车搜索
	                    $('.input_enter').keydown(function(e){
	                        if(e.keyCode==13){
	                            $('#searchBtn').trigger('click');

	                        }
	                    });
					},
					//页面按钮事件绑定
					/* 导航的三个按钮 编辑 添加 删除 */
					editClick:function(){
						$('#editPage').find('.u-msg-title').html("编辑");
						viewModel.event.clearDt(viewModel.dtnew);
						var row = viewModel.dt1.getSelectedRows()[0];
						if(row){
							viewModel.dtnew.setSimpleData(viewModel.dt1.getSimpleData({type: 'select'}));
							window.md = u.dialog({id: 'editDialog', content: '#editPage', hasCloseMenu: true});
						}else{
							u.messageDialog({msg:'请选择要编辑的数据！',title:'操作提示',btnText:'确定'});
						}
					},
					addClick:function(){
						$('#editPage').find('.u-msg-title').html("新增");
						viewModel.event.clearDt(viewModel.dtnew);
						var newr = viewModel.dtnew.createEmptyRow();
						newr.setValue("radiodatacontroller","Y");
						newr.setValue("radiodataassociate","Y");
						viewModel.dtnew.setRowSelect(newr);
						window.md = u.dialog({id: 'addDialog', content: '#editPage', hasCloseMenu: true});
						$('#addDialog').css('width', '70%');
					},
					delClick:function(){
						var row = viewModel.dt1.getSelectedRows()[0];
						if(row){
							var data = viewModel.dt1.getSelectedRows()[0].getSimpleData()
							u.confirmDialog({
					            msg: "是否删除"+data.name+"?",
					            title: "删除确认",
					            onOk: function () {
					                viewModel.event.deleteData(data);
					                viewModel.dt1.removeRow(viewModel.dt1.getSelectedIndexs());
					            },
					            onCancel: function () {
					            }
							});
						}else{
							u.messageDialog({msg:'请选择要删除的数据！',title:'操作提示',btnText:'确定'});
						}
					},
					searchClick:function(){
						viewModel.draw = 1; 
						viewModel.event.initCardTableList();
					},
					saveOkClick:function(){
						var data = viewModel.dtnew.getSimpleData()[viewModel.dtnew.getSelectedIndexs()];
	                    if (!viewModel.app.compsValidate($('#editPage')[0])) {
	                        return;
	                    }
						viewModel.event.saveData(data);
					},
					saveCancelClick:function(e){
							md.close();
					}
				}


#### 增删改查代码说明

增删改查的实现都分为三个步骤,1、实现对前台按钮的响应。2、实现前台按钮按下后的处理逻辑。3、调用相应的ajax请求。以卡片表为例，其中initCardTableList、deleteData、saveData为ajax请求发送方法。editClick、addClick、delClick、为对前台按钮按下的响应方法。searchClick、saveOkClick、saveCancelClick为响应菜单弹出后，各种不同处理方法、这些方法分别调用不同的ajax请求发送方法。

源码如下：

**ajax请求发送方法**
其中initCardTableList读取

					// 卡片表数据读取
					initCardTableList:function(){
						var jsonData={
								pageIndex:viewModel.draw-1,
								pageSize:viewModel.pageSize,
								sortField:"createtime",
								sortDirection:"asc"
						};
						$(element).find("#search").each(function(){
							if(this.value == undefined || this.value =='' || this.value.length ==0 ){
								//不执行操作
							}else{
								jsonData['search_searchParam'] =  this.value.replace(/(^\s*)|(\s*$)/g, "");  //去掉空格
							}
						});
						$.ajax({
							type:'get',
							url:listUrl,
							datatype:'json',
							data:jsonData,
							contentType: 'application/json;charset=utf-8',
							success:function(res){
								if(res){
									if( res.success =='success'){
										if(res.detailMsg.data){
											viewModel.totleCount=res.detailMsg.data.totalElements;
											viewModel.totlePage=res.detailMsg.data.totalPages;
											viewModel.event.comps.update({totalPages:viewModel.totlePage,pageSize:viewModel.pageSize,currentPage:viewModel.draw,totalCount:viewModel.totleCount});
											viewModel.dt1.removeAllRows();
											viewModel.dt1.clear();
											viewModel.dt1.setSimpleData(res.detailMsg.data.content,{unSelect:true});
										}
									}else{
										var msg = "";
										if(res.success == 'fail_global'){
											msg = res.message;
										}else{
											for (var key in res.detailMsg) {
												msg += res.detailMsg[key] + "<br/>";
											}
										}
										u.messageDialog({msg:msg,title:'请求错误',btnText:'确定'});
									}
								}else{
									u.messageDialog({msg:'后台返回数据格式有误，请联系管理员',title:'数据错误',btnText:'确定'});
								}
							},
							error:function(er){
								u.messageDialog({msg:'请求超时',title:'请求错误',btnText:'确定'});
							}
						});
					},

deleteDatas删除方法

						//删除方法
						deleteData: function(data) {
							var datalist = viewModel.event.genDataList(data);
							var json = JSON.stringify(datalist);
							$.ajax({
								url: delUrl,
								data: json,
								dataType: 'json',
								type: 'post',
								contentType: 'application/json',
								success: function (res) {
									//md.close();
									if(res){
										if (res.success == 'success') {
											u.messageDialog({msg:'删除成功',title:'操作提示',btnText:'确定'});
										} else {
											var msg = "";
											for(var key in res.message){
												msg +=res.message[key]+"<br/>";
											}
											u.messageDialog({msg:'msg',title:'操作提示',btnText:'确定'});
										}
									}else{
										u.messageDialog({msg:'无返回数据',title:'操作提示',btnText:'确定'});
									}
									
								},
								error:function(er){
									u.messageDialog({msg:'操作请求失败，'+er,title:'操作提示',btnText:'确定'});
								}
							});
						},

saveData新增和更新方法

					//新增和更新方法
					saveData:function(data) {
						var datalist = viewModel.event.genDataList(data);
						var json = JSON.stringify(datalist);
						$.ajax({
							url: saveUrl,
							type: 'post',
							data: json,
							dataType: 'json',
							contentType: 'application/json',
							success: function (res) {
								if(res){
									if( res.success =='success'){
			                            viewModel.event.initCardTableList();
										u.messageDialog({msg:'操作成功',title:'操作提示',btnText:'确定'});
										md.close();
									}else{
										var msg = "";
										if(res.success == 'fail_global'){
											msg = res.message;
										}else{
											for (var key in res.detailMsg) {
												msg += res.detailMsg[key] + "<br/>";
											}
										}
										u.messageDialog({msg:msg,title:'操作提示',btnText:'确定'});
									}
								}else{
									u.messageDialog({msg:'没有返回数据',title:'操作提示',btnText:'确定'});
								}
							}
						});
					},

**前台按钮响应方法**

editClick编辑按钮响应

					editClick:function(){
						$('#editPage').find('.u-msg-title').html("编辑");
						viewModel.event.clearDt(viewModel.dtnew);
						var row = viewModel.dt1.getSelectedRows()[0];
						if(row){
							viewModel.dtnew.setSimpleData(viewModel.dt1.getSimpleData({type: 'select'}));
							window.md = u.dialog({id: 'editDialog', content: '#editPage', hasCloseMenu: true});
						}else{
							u.messageDialog({msg:'请选择要编辑的数据！',title:'操作提示',btnText:'确定'});
						}
					},

addClick添加按钮响应

					addClick:function(){
						$('#editPage').find('.u-msg-title').html("新增");
						viewModel.event.clearDt(viewModel.dtnew);
						var newr = viewModel.dtnew.createEmptyRow();
						newr.setValue("radiodatacontroller","Y");
						newr.setValue("radiodataassociate","Y");
						viewModel.dtnew.setRowSelect(newr);
						window.md = u.dialog({id: 'addDialog', content: '#editPage', hasCloseMenu: true});
						$('#addDialog').css('width', '70%');
					},

delClick删除按钮响应

					delClick:function(){
						var row = viewModel.dt1.getSelectedRows()[0];
						if(row){
							var data = viewModel.dt1.getSelectedRows()[0].getSimpleData()
							u.confirmDialog({
					            msg: "是否删除"+data.name+"?",
					            title: "删除确认",
					            onOk: function () {
					                viewModel.event.deleteData(data);
					                viewModel.dt1.removeRow(viewModel.dt1.getSelectedIndexs());
					            },
					            onCancel: function () {
					            }
							});
						}else{
							u.messageDialog({msg:'请选择要删除的数据！',title:'操作提示',btnText:'确定'});
						}
					},


**处理逻辑调用方法**

searchClick搜索逻辑

					searchClick:function(){
						viewModel.draw = 1; 
						viewModel.event.initCardTableList();
					},

saveOkClick保存逻辑

					saveOkClick:function(){
						var data = viewModel.dtnew.getSimpleData()[viewModel.dtnew.getSelectedIndexs()];
	                    if (!viewModel.app.compsValidate($('#editPage')[0])) {
	                        return;
	                    }
						viewModel.event.saveData(data);
					},

saveCancelClick保存取消按钮逻辑

					saveCancelClick:function(e){
							md.close();
					}

### html
增删改查片段说明


### 按钮事件的绑定

            <div class="u-button-group margin-0" style="float:right;">
                <button class="u-button u-button-primary "
                        id="addButton" data-bind="click: event.addClick">新增
                </button>
                <button class="u-button u-button-primary "
                        id="editButton" data-bind="click: event.editClick">编辑
                </button>
                <button class="u-button u-button-primary" id="delButton" data-bind="click: event.delClick">删除</button>
            </div>

### 列表显示的绑定

    <!--table-->
    <div class="main-container cartable-container">
        <div class="hr-table ">
            <div class="u-table b-table width-full"
                 u-meta='{"id":"gridcardtable","type":"grid","data":"dt1","columnWidth":"150px"}'>
                <div options='{"field":"code","title":"系统编码","dataType":"String","editType":"string"}'></div>
                <div options='{"field":"name","title":"系统名称","dataType":"String","editType":"string"}'></div>
                <div options='{"field":"gateway","title":"网关地址","dataType":"String","editType":"string"}'></div>
                <div options='{"field":"radiodatacontroller","title":"需要授权","type":"u-checkbox","data":"dt1","editOptions":{"datasource":"radiodatacontroller"},"renderType":"comboRender"}'></div>
                <div options='{"field":"radiodataassociate","title":"用户自关联","type":"u-checkbox","data":"dt1","editOptions":{"datasource":"radiodataassociate"},"renderType":"comboRender"}'></div>
                <div options='{"field":"combodatamodel","title":"认证模式","dataType":"String","editType":"string","renderType":"comboRender","editOptions":{"datasource":"combodatamodel"}}'></div>
            </div>
        </div>
        <div id='pagination' class='pagination u-pagination pagination-sm'></div>
    </div>

###编辑弹出框的绑定
		
		<!-编辑-->
		<div class="" id="editPage" style="background: #fff; display: none;">
		    <div class="u-msg-title">新增</div>
		    <div class="u-msg-content " id="edit2">
		        <!--第一行-->
		        <div class=" u-row ">
		            <div class="u-col-2 ">
		                <label for="code" class="u-input-label right">系统编码:</label>
		            </div>
		            <div class="u-col-4 ">
		                <div class="u-input-group-before padding-left-10 "
		                     style="color: red;">*
		                </div>
		                <input type="text" id="code"
		                       class="u-form-control padding-left-15"
		                       u-meta='{"type":"string","data":"dtnew","field":"code"}'
		                       placeholder="系统编码不能为空">
		            </div>
		            <div class="u-col-2 right-col">
		                <label class="u-input-label right">系统名称:</label>
		            </div>
		            <div class="u-col-4 ">
		                <div class="u-input-group-before padding-left-10 "
		                     style="color: red;">*
		                </div>
		                <input type="text" id="name"
		                       class="u-form-control padding-left-15"
		                       u-meta='{"type":"string","data":"dtnew","field":"name"}'
		                       placeholder="系统名称不能为空">
		            </div>
		        </div>
		        <!--第二行-->
		        <div class=" system-row u-row margin-top-20">
		            <div class="u-col-2 ">
		                <label class="u-input-label right">网关地址:</label>
		            </div>
		            <div class="u-col-4 ">
		                <input type="text" id="gateway" class="u-form-control"
		                       u-meta='{"id":"gateway","type":"string","data":"dtnew","field":"gateway"}'
		                       placeholder="网关地址">
		            </div>
		            <div class="u-col-2 right-col ">
		                <label class="u-input-label right">认证模式:</label>
		            </div>
		            <div id="model" class="u-col-4 u-combo u-label-floating"
		                 u-meta='{"id":"model","type":"u-combobox","data":"dtnew","field":"combodatamodel","renderType":"comboRender","datasource":"combodatamodel"}'>
		                <input class="u-input"/> <span class="u-combo-icon"></span>
		            </div>
		        </div>
		
		        <!--第三行-->
		        <div class=" system-row u-row margin-top-20">
		            <div class="u-col-2">
		                <label class="u-input-label right">需要授权: </label>
		            </div>
		            <div class="u-col-4" id="underController"
		                 u-meta='{"id":"underController","type":"u-radio","data":"dtnew","field":"radiodatacontroller","renderType":"radioRender","datasource":"radiodatacontroller"}'>
		                <div>
		                    <label class="u-radio"> <input type="radio" class="u-radio-button" name="radiodatacontroller"> <span
		                            class="u-radio-label"></span>
		                    </label>
		                </div>
		            </div>
		        </div>
		
		        <!--第四行-->
		        <div class=" system-row u-row margin-top-20">
		            <div class="u-col-2">
		                <label class="u-input-label right">自行关联: </label>
		            </div>
		            <div class="u-col-4" id="userAssociate"
		                 u-meta='{"id":"userAssociate","type":"u-radio","data":"dtnew","field":"radiodataassociate","renderType":"radioRender","datasource":"radiodataassociate"}'>
		                <div>
		                    <label class="u-radio"> <input type="radio"
		                                                   class="u-radio-button" name="radiodataassociate"> <span
		                            class="u-radio-label"></span>
		                    </label>
		                </div>
		            </div>
		        </div>
		    </div>
		    <div class=" u-msg-footer ">
		        <div class="pull-right">
		            <button type="button" class="u-button u-button-white editCancel margin-right-15 "
		                    id="editCancel" data-bind="click: event.saveCancelClick">取消
		            </button>
		            <button type="button"
		                    class="u-button raised u-button-primary margin-right-10" id="editOk"
		                    data-bind="click: event.saveOkClick">保存
		            </button>
		
		        </div>
		    </div>
		</div>

## 菜单添加以及路由配置

index.html添加节点

            <li class='nav-li'>
                <a href="#cardtable/cardtable">
                    <i class="uf uf-4-s-o-2"></i>
                    <span class="nav-title">卡片表</span>
                </a>
            </li>

## 调试页面

页面截图

![image](/articles/iuap-develop/4-/images/htmljs.jpg)
