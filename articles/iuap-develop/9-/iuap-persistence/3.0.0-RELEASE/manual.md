# 持久化组件 #

## 业务需求 ##
业务应用在快速搭建过程中大多数都需要持久化，需要提供一种简单的持久化集成方式，快速搭建支持基础CRUD的环境，同时兼顾后期的扩展。持久化要简单易用，支持分页查询、保存和更新操作等。为了和其他iuap组件保持对Spring等第三方组件一致的版本依赖，需要平台提供基本的持久化、连接池等集成规范和示例。


## 解决方案 ##
iuap持久化组件提供对通用持久化功能的集成，包括Spring Data JPA、Mybatis、Spring JDBC等，同时集成数据库连接池Apache Tomcat-jdbc。本组件没有对代码进行封装，作用在于集成对基础的Spring Data JPA方式和Mybatis方式的依赖，保证iuap其他组件和持久化组件所依赖的三方组件版本统一。

工程搭建时，可以引入iuap持久化组件，快速的对持久化层进行集成，使用示例工程中提供的Spring配置文件模板，对工程的持久化方案进行筛选。


## 功能说明 ##

1.	集成Spring Data JPA；
2.	集成Mybatis；
3.	集成Spring JDBC；
4.	集成Tomcat JDBC数据库连接池；
5.	支持JPA分页查询；


# 整体设计 #

## 依赖环境 ##
- spring 4.0.5.RELEASE
- spring-data-jpa 1.6.0.RELEASE
- mybatis 3.3.0
- mybatis-spring 1.2.3
- hibernate-entitymanager 4.3.5.Final
- tomcat-jdbc  7.0.53

上述组件已经在iuap-persistence中依赖，不需要业务开发再次依赖，如果在集成前已经依赖，请自行处理版本冲突

## 注意事项 ##

- 默认使用Tomcat jdbc的连接池，为后续的动态数据源做基础
- 集成Mybatis的基础依赖，如果需要单独使用Mybatis，请参考iuap-mybatis组件
- 事务控制上依赖spring的事务管理，使用Spring-data-jpa时，请注意需要使用JpaTransactionManager
- 业务上如果对其中的持久化方式可以明确，需要删减示例中不需要的配置项

# 使用说明 #

## 配置方式 ##
**1.maven工程配置对持久化组件的依赖**

    <dependency>
        <groupId>com.yonyou.iuap</groupId>
        <artifactId>iuap-persistence</artifactId>
        <version>${iuap.modules.version}</version>
    </dependency>

${iuap.modules.version} 为在pom.xml中定义的需要引入组件的version。

**2.spring配置文件中，增加对连接池、数据源的依赖**

    <!-- 数据源配置, 使用Tomcat JDBC连接池 -->
    <bean id="dataSource" class="org.apache.tomcat.jdbc.pool.DataSource" destroy-method="close">
        <!-- Connection Info -->
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>

        <!-- Connection Pooling Info -->
        <property name="maxActive" value="${jdbc.pool.maxActive}"/>
        <property name="maxIdle" value="${jdbc.pool.maxIdle}"/>
        <property name="minIdle" value="0"/>
        <property name="maxWait" value="${jdbc.pool.maxWait}"/>
        <property name="defaultAutoCommit" value="true"/>

        <property name="minEvictableIdleTimeMillis" value="${jdbc.pool.minEvictableIdleTimeMillis}"/>
        <property name="removeAbandoned" value="${jdbc.pool.removeAbandoned}"/>
        <property name="removeAbandonedTimeout" value="${jdbc.pool.removeAbandonedTimeout}"/>
    </bean>

**3.如果使用Spring Data JPA方式进行持久化，spring配置文件中，增加如下配置**

    <!-- Jpa Entity Manager 配置 -->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="jpaVendorAdapter" ref="hibernateJpaVendorAdapter"/>
        <property name="packagesToScan" value="com.yonyou.iuap.entity"/>
        <property name="jpaProperties">
            <props>
                <!-- 命名规则 My_NAME->MyName -->
                <prop key="hibernate.ejb.naming_strategy">org.hibernate.cfg.ImprovedNamingStrategy</prop>
                <prop key="hibernate.show_sql">true</prop>
            </props>
        </property>
    </bean>

    <bean id="hibernateJpaVendorAdapter" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
        <property name="databasePlatform">
            <bean factory-method="getDialect" class="org.springside.modules.persistence.Hibernates">
                <constructor-arg ref="dataSource"/>
            </bean>
        </property>
    </bean>

    <!-- Jpa 事务配置 -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>

    <!-- Spring Data Jpa配置 -->
    <jpa:repositories base-package="com.yonyou.iuap.repository"
                      transaction-manager-ref="transactionManager"
                      entity-manager-factory-ref="entityManagerFactory"/>

**4.如果项目中使用Mybatis方式进行持久化，增加Mybatis的配置**

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

**5.增加对事务的控制**

    <!-- 使用annotation定义事务 -->
    <aop:aspectj-autoproxy/>

    <tx:annotation-driven transaction-manager="transactionManager"/>

    <!-- 注意！！！单独使用mybatis时候事务配置可以采用DataSourceTransactionManager,如果和jpa一起使用，配置JpaTransactionManager -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

**6.JPA方式接口示例**

	public interface DemoEntityJpaDao extends PagingAndSortingRepository<DemoEntity, String> , JpaSpecificationExecutor<DemoEntity>

**7.Mybatis方式接口示例**

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


**8.对应service的业务方法上，增加事务注解**

	@Transactional
	public int complexQuotationOperate(IpuQuotation quotation){

	//事务的传播机制根据用户的需求来配置，如新启子事务
	@Transactional(propagation=Propagation.REQUIRES_NEW)
	
**9:更多API操作和配置方式，请参考对应的示例工程(DevTool/examples/example\_iuap\_persistence)**


## Sping Data JPA代码示例 ##

导入已有的Maven工程iuap\_persistence\_example,里面有简单的示例：

- 实体类示例

		@Entity
		@Table(name="example_demo")
		@NamedQuery(name="DemoEntity.findAll", query="SELECT d FROM DemoEntity d")
		public class DemoEntity implements Serializable {
			private static final long serialVersionUID = 1L;
		
			@Id
			@Column(name="id")
			private String id;
			
			@NotBlank(message="测试编码不能为空!")
			private String code;
		
			private String memo;
		
			private String name;
			
			private String isdefault;
		
			public DemoEntity() {
			}
			public String getId() {
				return id;
			}
			public void setId(String id) {
				this.id = id;
			}
			public String getCode() {
				return this.code;
			}
			public void setCode(String code) {
				this.code = code;
			}
			public String getMemo() {
				return this.memo;
			}
			public void setMemo(String memo) {
				this.memo = memo;
			}
			public String getName() {
				return this.name;
			}
			public String getIsdefault() {
				return isdefault;
			}
			public void setIsdefault(String isdefault) {
				this.isdefault = isdefault;
			}
			public void setName(String name) {
				this.name = name;
			}
		}

- 数据库操作类示例（只需要定义接口）

		public interface DemoEntityJpaDao extends PagingAndSortingRepository<DemoEntity, String> , JpaSpecificationExecutor<DemoEntity>{
		
			DemoEntity findByCode(String code);
			
			@Query("select d from DemoEntity d where code in (:codes)")
			List<DemoEntity> findByConditions(String[] codes);
			
			@Query(value = "select * from example_demo where id = ?1", nativeQuery=true)
			DemoEntity getDemoByNativeSql(String id);
			
			@Modifying
			@Query(value = "delete from example_demo where id = ?1", nativeQuery=true)
			void deleteDemoByIdWithNativeSql(String id);
		}

- 服务类示例

		@Service
		public class DemoService {
		
		    @Autowired
		    private DemoEntityJpaDao dao;
		    
			@Autowired
			private JdbcTemplate jt;
		
		    public DemoEntity getDemoById(String id) {
		        return dao.findOne(id);
		    }
		
		    public DemoEntity getDemoBySql(String id) {
		        return dao.getDemoByNativeSql(id);
		    }
		    
		    @Transactional
		    public void deleteById(String id) {
		    	dao.delete(id);
		    }
		
		    @Transactional
		    public void deleteDemoByIdUseSql(String id) {
		        dao.deleteDemoByIdWithNativeSql(id);
		    }
		
		    public DemoEntity saveEntity(DemoEntity entity) throws SQLException {
		        if (StringUtils.isBlank(entity.getId())) {
		            entity.setId(UUID.randomUUID().toString());
		        }
		        entity = dao.save(entity);
		        return entity;
		    }
		
		    public Page<DemoEntity> getDemoPage(Map<String, Object> searchParams, PageRequest pageRequest) {
		        Specification<DemoEntity> spec = buildSpecification(searchParams);
		        return dao.findAll(spec, pageRequest);
		    }
		
		    /**
		     * 创建动态查询条件组合.
		     */
		    public Specification<DemoEntity> buildSpecification(Map<String, Object> searchParams) {
		        Map<String, SearchFilter> filters = SearchFilter.parse(searchParams);
		        Specification<DemoEntity> spec = DynamicSpecifications.bySearchFilter(filters.values(), DemoEntity.class);
		        return spec;
		    }
		}

## Mybatis代码示例 ##

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

