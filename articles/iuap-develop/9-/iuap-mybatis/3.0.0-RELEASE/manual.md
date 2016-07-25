# Mybatis持久化组件 #

## 业务需求 ##

在业务应用的持久化工作中，有时会选用MyBatis作为持久化框架支持。MyBatis的Sql语句写在XML配置文件里，将Sql语句与程序代码解耦，便于统一管理和优化。但是MyBatis并没有完善的分页支持，需要业务自己来实现。

## 解决方案 ##

iuap平台使用iuap-mybatis作为MyBatis持久化的支持并提供了统一的Spring扫描注解和分页插件来支持分页的实现。

在前面的iuap-persistence组件中已经对Mybatis的基础使用做了介绍说明，iuap-persistence同时依赖了基础的Spring Data JPA和Mybatis，如果项目上明确只使用Mybatis，则不需要引入iuap-persistence组件，直接依赖iuap-mybatis组件即可，注意要和iuap其他组件的版本保持一致。


## 功能说明 ##
1.	提供统一的Spring扫描注解；
2.	提供Mybatis分页插件;


# 整体设计 #

## 依赖环境 ##

- spring 4.0.5.RELEASE
- mybatis 3.3.0
- mybatis-spring 1.2.3

## 功能特性 ##
iuap-mybatis提供了统一的Spring扫描注解和分页插件。通过@MyBatisRepository确定mybatis提供服务的接口，再使用mybatis的xml文件管理实现mybatis服务的sql语句。

分页插件支持MyBatis3.2.0~3.3.0(包含)

# 使用说明 #

## Maven依赖配置 ##

    	<dependency>
    		<groupId>com.yonyou.iuap</groupId>
    		<artifactId>iuap-mybatis</artifactId>
    		<version>${iuap.modules.version}</version>
    	</dependency>

## 简单示例 ##

**实体类**
    
    public class DemoEntity implements Serializable {
    
    	private static final long serialVersionUID = 6794214450179665569L;
    
    	private String id;
    	private String name;
    	private Integer age;
    	public String getId() {
    		return id;
    	}
    	public void setId(String id) {
    		this.id = id;
    	}
    	public String getName() {
    		return name;
    	}
    	public void setName(String name) {
    		this.name = name;
    	}
    	public Integer getAge() {
    		return age;
    	}
    	public void setAge(Integer age) {
    		this.age = age;
    	}
    	public static long getSerialversionuid() {
    		return serialVersionUID;
    	}
    	
    }
    
**接口**

    //使用@MyBatisRepository注解,确定服务接口
    @MyBatisRepository
    public interface DemoEntityMapper {
    	public DemoEntity findPeploeByID(String ID);
    }
    

**mybatis xml文件**

- 与接口类同名的xml文件DemoEntityMapper.xml
	        
	    <?xml version="1.0" encoding="UTF-8"?>
	    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	    <mapper namespace="com.zhu.example.interfaces.DemoEntityMapper">
	    	<resultMap id="BaseResultMap" type="com.zhu.example.entity.DemoEntity">
	    		<id column="id" property="id" jdbcType="CHAR" />
	    		<result column="name" property="name" jdbcType="CHAR" />
	    		<result column="age" property="age" jdbcType="VARCHAR" />
	    	</resultMap>
	    
	    	<sql id="Base_Column_List">
	    		id,name,age
	    	</sql>
	    	
	    	<select id="findPeploeByID"  parameterType="java.lang.String" resultMap="BaseResultMap">
	    		select
	    		<include refid="Base_Column_List" />
	    		from my_entity
	    		where id = #{id}
	    	</select>
	    	
	    </mapper>



## 配置文件设置 ##

	    <!-- 使用annotation定义事务 -->
	    <aop:aspectj-autoproxy/>
	
		<!-- 单独使用mybatis时候，可以使用DataSourceTransactionManager -->
	    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	        <property name="dataSource" ref="dataSource"/>
	    </bean>
	
	    <tx:annotation-driven transaction-manager="transactionManager"/>
	
	    <!-- MyBatis配置 -->
	    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	        <property name="dataSource" ref="dataSource"/>
	        <!-- 自动扫描entity目录, 省掉Configuration.xml里的手工配置 -->
	        <property name="typeAliasesPackage" value="com.yonyou.iuap.entity"/>
	        <!-- 显式指定Mapper文件位置 -->
	        <property name="mapperLocations" value="classpath:/mybatis/**/*Mapper.xml"/>
	        <property name="plugins">
	            <array>
	                <bean id="paginationInterceptor"
	                      class="com.yonyou.iuap.persistence.mybatis.plugins.PaginationInterceptor">
	                    <property name="properties">
	                        <props>
	                            <prop key="dbms">mysql</prop>
	                            <prop key="sqlRegex">.*retrievePage.*</prop>
	                        </props>
	                    </property>
	                </bean>
	            </array>
	        </property>
	    </bean>
	    <!-- 扫描basePackage下所有以@MyBatisRepository标识的 接口-->
	    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	        <property name="basePackage" value="com.yonyou.iuap.repository"/>
	        <property name="annotationClass" value="com.yonyou.iuap.persistence.mybatis.anotation.MyBatisRepository"/>
	        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
	    </bean>
    



## 示例工程说明 ##

开发工具包中包含对iuap-mybatis组件的使用示例(DevTool/examples/example-iuap-mybatis),请参考示例工程中的具体配置和使用示例。

- 实体类示例
	
		public class IpuQuotation implements Serializable {
		
			private static final long serialVersionUID = -9169987729255268660L;
		
			private String ipuquotaionid;
		
		    private Date buyoffertime;
		
		    private String contact;
		
		    private Date createtime;
		
		    private String description;
		
		    private String phone;
		
		    private Date processtime;
		
		    private String processor;
		
		    private Date qtexpiredate;
		
		    private String subject;
		
		    private Date ts;
		
		    private Short dr;
		
		    private List<IpuQuotationDetail> datailentitylist;
		
		    public String getIpuquotaionid() {
		        return ipuquotaionid;
		    }
		    public void setIpuquotaionid(String ipuquotaionid) {
		        this.ipuquotaionid = ipuquotaionid == null ? null : ipuquotaionid.trim();
		    }		
		    public Date getBuyoffertime() {
		        return buyoffertime;
		    }	
		    public void setBuyoffertime(Date buyoffertime) {
		        this.buyoffertime = buyoffertime;
		    }
		
			... ...
		
		    public List<IpuQuotationDetail> getDatailentitylist() {
		        return datailentitylist;
		    }
		
		    public void setDatailentitylist(List<IpuQuotationDetail> datailentitylist) {
		        this.datailentitylist = datailentitylist;
		    }
		}

- 数据库操作类示例
	
			@MyBatisRepository
			public interface IpuQuotationMapper extends PageMapper<IpuQuotation> {
			
			    int deleteByPrimaryKey(String id);
			
			    int insert(IpuQuotation record);
			
			    IpuQuotation selectByPrimaryKey(String id);
			
			    IpuQuotation selectChildrenByPrimaryKey(String id);
			
			    int updateByPrimaryKeySelective(IpuQuotation record);
			
			    int updateByPrimaryKey(IpuQuotation record);
			
			    PageResult<IpuQuotation> retrievePage(PageRequest pageRequest, @Param("subject") String subject);
			}


- 映射文件示例(注意spring配置文件中对mybatis映射文件的位置)
	
			<?xml version="1.0" encoding="UTF-8"?>
			<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
			<mapper namespace="com.yonyou.iuap.repository.quotation.IpuQuotationMapper">
			    <resultMap id="BaseResultMap" type="com.yonyou.iuap.entity.quotation.IpuQuotation">
			        <id column="ipuquotaionid" property="ipuquotaionid" jdbcType="CHAR"/>
			        <result column="buyoffertime" property="buyoffertime" jdbcType="TIMESTAMP"/>
			        <result column="contact" property="contact" jdbcType="VARCHAR"/>
			        <result column="createtime" property="createtime" jdbcType="TIMESTAMP"/>
			        <result column="description" property="description" jdbcType="VARCHAR"/>
			        <result column="phone" property="phone" jdbcType="VARCHAR"/>
			        <result column="processtime" property="processtime" jdbcType="TIMESTAMP"/>
			        <result column="processor" property="processor" jdbcType="VARCHAR"/>
			        <result column="qtexpiredate" property="qtexpiredate" jdbcType="TIMESTAMP"/>
			        <result column="subject" property="subject" jdbcType="VARCHAR"/>
			        <result column="ts" property="ts" jdbcType="TIMESTAMP"/>
			        <result column="dr" property="dr" jdbcType="SMALLINT"/>
			    </resultMap>
			
			    <resultMap id="childrenMap" type="com.yonyou.iuap.entity.quotation.IpuQuotation">
			        <id column="ipuquotaionid" property="ipuquotaionid" jdbcType="CHAR"/>
			        <result column="buyoffertime" property="buyoffertime" jdbcType="TIMESTAMP"/>
			        <result column="contact" property="contact" jdbcType="VARCHAR"/>
			        <result column="createtime" property="createtime" jdbcType="TIMESTAMP"/>
			        <result column="description" property="description" jdbcType="VARCHAR"/>
			        <result column="phone" property="phone" jdbcType="VARCHAR"/>
			        <result column="processtime" property="processtime" jdbcType="TIMESTAMP"/>
			        <result column="processor" property="processor" jdbcType="VARCHAR"/>
			        <result column="qtexpiredate" property="qtexpiredate" jdbcType="TIMESTAMP"/>
			        <result column="subject" property="subject" jdbcType="VARCHAR"/>
			        <result column="ts" property="ts" jdbcType="TIMESTAMP"/>
			        <result column="dr" property="dr" jdbcType="SMALLINT"/>
			        <collection property="datailentitylist" ofType="com.yonyou.iuap.entity.quotation.IpuQuotationDetail">
			            <id column="c_id" property="ipuquotationdetailid" jdbcType="CHAR"/>
			            <result column="c_productdesc" property="productdesc" jdbcType="VARCHAR"/>
			            <result column="c_productname" property="productname" jdbcType="VARCHAR"/>
			            <result column="c_purchaseamount" property="purchaseamount" jdbcType="NUMERIC"/>
			            <result column="c_unit" property="unit" jdbcType="VARCHAR"/>
			            <result column="c_ipuquotaionid" property="ipuquotaionid" jdbcType="CHAR"/>
			            <result column="c_ts" property="ts" jdbcType="TIMESTAMP"/>
			            <result column="c_dr" property="dr" jdbcType="SMALLINT"/>
			        </collection>
			    </resultMap>
			    <sql id="Base_Column_List">
			        ipuquotaionid, buyoffertime, contact, createtime, description, phone, processtime,
			        processor, qtexpiredate, subject, ts, dr
			    </sql>
			    <sql id="oneToManyColumn">
			        t.ipuquotaionid, t.buyoffertime,
			        t.contact, t.createtime, t.description,
			        t.phone, t.processtime, t.processor, t.qtexpiredate, t.subject, t.ts, t.dr,
			        tc.ipuquotationdetailid as c_id, tc.productdesc as c_productdesc,tc.productname as c_productname,
			        tc.purchaseamount as c_purchaseamount, tc.unit as c_unit,
			        tc.ipuquotaionid as c_ipuquotaionid, tc.ts as c_ts,tc.dr as c_dr
			    </sql>
			    <select id="selectByPrimaryKey" parameterType="java.lang.String" resultMap="BaseResultMap">
			        select
			        <include refid="Base_Column_List"/>
			        from ipuquotation
			        where ipuquotaionid = #{ipuquotaionid}
			    </select>
			
			    <select id="selectChildrenByPrimaryKey" parameterType="java.lang.String" resultMap="childrenMap">
			        select
			        <include refid="oneToManyColumn"/>
			        from ipuquotation t LEFT OUTER JOIN ipuquotationdetail tc on t.ipuquotaionid=tc.ipuquotaionid
			        where t.ipuquotaionid = #{ipuquotaionid}
			    </select>
			
			    <select id="count" parameterType="map" resultType="int">
			        select
			        count(*)
			        from ipuquotation
			        where 1=1
			        <if test="subject != null and subject!=''">
			            and subject like concat('%',#{subject})
			        </if>
			    </select>
			
			    <select id="queryPage" parameterType="map" resultMap="BaseResultMap">
			        select
			        <include refid="Base_Column_List"/>
			        from ipuquotation
			        where 1=1
			        <if test="subject != null and subject!=''">
			            and subject like concat('%',#{subject})
			        </if>
			        <if test="page.sort!=null">
			            order by
			            <foreach collection="page.sort" item="item" separator=" ">
			                ${item.property} ${item.direction}
			            </foreach>
			        </if>
			        limit #{page.size} offset #{page.offset}
			    </select>
			
			    <delete id="deleteByPrimaryKey" parameterType="java.lang.String">
			    delete from ipuquotation
			    where ipuquotaionid = #{ipuquotaionid}
			  </delete>
			    <insert id="insert" parameterType="com.yonyou.iuap.entity.quotation.IpuQuotation">
			        insert into ipuquotation (ipuquotaionid, buyoffertime, contact,
			          createtime, description, phone,
			          processtime, processor, qtexpiredate,
			          subject, ts, dr)
			        values (#{ipuquotaionid,jdbcType=CHAR}, #{buyoffertime,jdbcType=TIMESTAMP}, #{contact,jdbcType=VARCHAR},
			          #{createtime,jdbcType=TIMESTAMP}, #{description,jdbcType=VARCHAR}, #{phone,jdbcType=VARCHAR},
			          #{processtime,jdbcType=TIMESTAMP}, #{processor,jdbcType=VARCHAR}, #{qtexpiredate,jdbcType=TIMESTAMP},
			          #{subject,jdbcType=VARCHAR}, #{ts,jdbcType=TIMESTAMP}, #{dr,jdbcType=SMALLINT})
			    </insert>
			    <update id="updateByPrimaryKeySelective" parameterType="com.yonyou.iuap.entity.quotation.IpuQuotation">
			        update ipuquotation
			        <set>
			            <if test="buyoffertime != null">
			                buyoffertime = #{buyoffertime,jdbcType=TIMESTAMP},
			            </if>
			            <if test="contact != null">
			                contact = #{contact,jdbcType=VARCHAR},
			            </if>
			            <if test="createtime != null">
			                createtime = #{createtime,jdbcType=TIMESTAMP},
			            </if>
			            <if test="description != null">
			                description = #{description,jdbcType=VARCHAR},
			            </if>
			            <if test="phone != null">
			                phone = #{phone,jdbcType=VARCHAR},
			            </if>
			            <if test="processtime != null">
			                processtime = #{processtime,jdbcType=TIMESTAMP},
			            </if>
			            <if test="processor != null">
			                processor = #{processor,jdbcType=VARCHAR},
			            </if>
			            <if test="qtexpiredate != null">
			                qtexpiredate = #{qtexpiredate,jdbcType=TIMESTAMP},
			            </if>
			            <if test="subject != null">
			                subject = #{subject,jdbcType=VARCHAR},
			            </if>
			            <if test="ts != null">
			                ts = #{ts,jdbcType=TIMESTAMP},
			            </if>
			            <if test="dr != null">
			                dr = #{dr,jdbcType=SMALLINT},
			            </if>
			        </set>
			        where ipuquotaionid = #{ipuquotaionid,jdbcType=CHAR}
			    </update>
			    <update id="updateByPrimaryKey" parameterType="com.yonyou.iuap.entity.quotation.IpuQuotation">
			        update ipuquotation
			        set buyoffertime = #{buyoffertime,jdbcType=TIMESTAMP},
			          contact = #{contact,jdbcType=VARCHAR},
			          createtime = #{createtime,jdbcType=TIMESTAMP},
			          description = #{description,jdbcType=VARCHAR},
			          phone = #{phone,jdbcType=VARCHAR},
			          processtime = #{processtime,jdbcType=TIMESTAMP},
			          processor = #{processor,jdbcType=VARCHAR},
			          qtexpiredate = #{qtexpiredate,jdbcType=TIMESTAMP},
			          subject = #{subject,jdbcType=VARCHAR},
			          ts = #{ts,jdbcType=TIMESTAMP},
			          dr = #{dr,jdbcType=SMALLINT}
			        where ipuquotaionid = #{ipuquotaionid,jdbcType=CHAR}
			    </update>
			
			    <select id="retrievePage" parameterType="map" resultMap="BaseResultMap">
			        select
			        <include refid="Base_Column_List"/>
			        from ipuquotation
			        where 1=1
			        <if test="subject != null and subject!=''">
			            and subject like concat(#{subject},'%')
			        </if>
			    </select>
			</mapper>

## 分页支持说明 ##

该插件目前支持以下数据库的<b>物理分页</b>:

 1. `Oracle`
 2. `Mysql`

配置`dialect`属性时，可以使用小写形式：

`oracle`,`mysql`


## 版本迁移 ##

如果您正在使用其他分页插件，想迁移到iuap-mybatis分页插件，iuap-mybatis为您提供了方便的迁移方案。

- 查询条件转换接口 `RequestConvertor`
  应用通过该接口实现类，将自定义的查询条件转换为iuap-mybatis插件的分页条件，包括数据获取区间，排序条件等等。
  
		    <property name="requestConvertor">
		        <bean class="xxx.xxx.xxxRequestConvertor"/>
		    </property>

- 查询结果转换接口 `ResultConvertor`
  应用通过实现该接口，将iuap-mybatis的返回结果PageResult转换为应用自己的page对象。
	
	         <property name="resultConvertor">
	             <bean class="xxx.xxx.xxxResultConvertor"/>
	         </property>
