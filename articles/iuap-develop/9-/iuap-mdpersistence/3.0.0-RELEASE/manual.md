# 元数据持久化组件概述 #

## 业务需求 ##

业务系统在进行模型关系使用和查询时，不可避免的需要对数据库表有深入了解，知道具体字段的含义，且开发效率差，简单查询都需要代码开发
运行效率查，公式解析解析代价


## 解决方案##

iuap-mdjdbc组件提供一种完全基于模型角度，不用了解数据库结构的支持复杂条件查询的访问方式，有效的提高数据访问效率，且切合业务实际。

## 功能说明##

1.	基于元数据，支持对业务数据的增、删、改、查；
2.	支持适用于多租户应用架构的多数据源配置与访问；
3.	支持基于元数据操作业务数据时的数据变化事件，默认提供日志记录；
4.	支持基于元数据的业务数据查询缓存，同时支持自定义业务数据缓存；
5.	支持SQL脚本的代理执行和自定义接口扩展。

# 整体设计 #

## 依赖环境 ##
组件采用Maven进行编译和打包发布，依赖spring框架,引入了UAP平台的一些基础组件如iuap-mdpersistence、iuap-mdperspi、iuap-log和iuap-cache、iuap-mybatis，以及一些基础组件如tomcat-jdbc、mysql-connector-java、commons-collections4，其对外提供的依赖方式如下：

业务系统在进行模型关系使用和查询时，不可避免的需要对数据库表有深入了解，知道具体字段的含义，且开发效率差，简单查询都需要代码开发
运行效率查，公式解析解析代价

##解决方案##

iuap-mdjdbc组件提供一种完全基于模型角度，不用了解数据库结构的支持复杂条件查询的访问方式，有效的提高数据访问效率，且切合业务实际。

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，依赖spring框架,引入了UAP平台的一些基础组件如iuap-mdpersistence、iuap-mdperspi、iuap-log和iuap-cache、iuap-mybatis，以及一些基础组件如tomcat-jdbc、mysql-connector-java、commons-collections4，其对外提供的依赖方式如下：

## 配置和使用方式 ##

### maven依赖 ###

```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-mdjdbc</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>
```


${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 流程说明 ##

请参照iuap-jdbc组件的流程说明

# 使用说明 #

## 组件包说明 ##

iuap元数据持久化组件是遵循元数据设计规范，基于iuap-jdbc的持久化组件。基于元数据的定义，该组件提供了对数据的增删改查，数据表扩展以及关联关系查询等功能。同时提供热点数据缓存，数据变更日志等功能。

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 流程说明 ##

请参照iuap-jdbc组件的流程说明

# 使用说明 #

## 组件包说明 ##

iuap元数据持久化组件是遵循元数据设计规范，基于iuap-jdbc的持久化组件。基于元数据的定义，该组件提供了对数据的增删改查，数据表扩展以及关联关系查询等功能。同时提供热点数据缓存，数据变更日志等功能。

##组件配置##


```
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
		http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd"
       default-lazy-init="true">
    <description>Spring公共配置</description>

    <!-- spring 事务配置 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="crossDBDataSource"/>
    </bean>

    <context:component-scan base-package="com.yonyou.metadata.mybatis.dao"/>
    <!-- 使用annotation定义事务 -->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>

    <context:property-placeholder system-properties-mode="OVERRIDE" ignore-unresolvable="true"
                                  location="application.properties"/>

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

    <!-- uapjdbc数据源，对接应用数据源 -->
    <bean id="crossDBDataSource" class="com.yonyou.iuap.persistence.bs.framework.ds.CrossdbDataSource"
          lazy-init="false">
        <!-- 此数据源对接应用的实际数据源 -->
        <constructor-arg name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 数据源配置,使用应用服务器的数据库连接池 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="crossDBDataSource"></property>
    </bean>
    
    <!-- 元数据持久化声明 -->
    <bean id="baseDAO" class="com.yonyou.iuap.persistence.bs.dao.MetadataDAO">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
        <property name="dbMetaHelper" ref="dbMetaInfo"/>
        
        <!-- 应用数据缓存管理器，如果不配置，应用数据不会被缓存 -->
        <property name="saasCacheManager" ref="saasCacheManager"/>
        
        <!-- 数据变更通知接口，如果不配置，不会触发数据变更通知 -->
        <property name="dataChangeNotifier" ref="dataChangeNotifier"/>
    </bean>
    
    <bean id="dbMetaInfo" class="com.yonyou.iuap.persistence.bs.util.MetadataDBMetaHelper">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>

    <bean id="redisPool" class="com.yonyou.iuap.cache.redis.RedisPoolFactory"
          scope="prototype" factory-method="createJedisPool">
        <constructor-arg value="${redis.url}"/>
    </bean>

    <bean id="jedisTemplate" class="org.springside.modules.nosql.redis.JedisTemplate">
        <constructor-arg ref="redisPool"></constructor-arg>
    </bean>

    <!-- 元数据持久化组件使用的缓存管理器 -->
    <bean id="saasCacheManager" class="com.yonyou.iuap.cache.SaasCacheManager">
        <property name="cacheManager">
            <bean class="com.yonyou.iuap.cache.CacheManager">
                <property name="jedisTemplate" ref="jedisTemplate"/>
                <property name="serializer">
                    <bean class="com.yonyou.iuap.persistence.bs.cache.serializer.MetaFastJsonSerializer">
                        <property name="metadataDBMetaHelper" ref="dbMetaInfo"/>
                    </bean>
                </property>
            </bean>
        </property>
    </bean>

    <!-- 数据变更接口通知器 -->
    <bean id="dataChangeNotifier" class="com.yonyou.iuap.persistence.bs.notifier.DataChangeNotifier">
    </bean>

	   <!-- 元数据服务调用声明 -->
	   <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 自动扫描entity目录, 省掉Configuration.xml里的手工配置 -->
        <property name="typeAliasesPackage"
                  value="com.yonyou.iuap.entity,com.yonyou.metadata.spi,com.yonyou.metadata.mybatis.util.publish.model"/>
        <!-- 显式指定Mapper文件位置 -->
        <property name="mapperLocations" value="classpath*:/mybatis/**/*.xml"/>
    </bean>
    <!-- 扫描basePackage下所有以@MyBatisRepository标识的 接口 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage"
                  value="com.yonyou.iuap.repository,com.yonyou.metadata.mybatis.dao"/>
        <property name="annotationClass"
                  value="com.yonyou.iuap.persistence.mybatis.anotation.MyBatisRepository"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>
    <bean class="com.yonyou.metadata.mybatis.service.MyBatisMetadataService">
    </bean>

    <!-- 元数据服务缓存配置 -->
    <bean id="metadataCache" class="com.yonyou.metadata.mybatis.util.MetadataCache">
        <property name="saasCacheMgr">
            <bean id="saasCacheManager" class="com.yonyou.iuap.cache.SaasCacheManager">
                <property name="cacheManager">
                    <bean class="com.yonyou.iuap.cache.CacheManager">
                        <property name="jedisTemplate" ref="jedisTemplate"/>
                    </bean>
                </property>
            </bean>
        </property>
        <property name="cacheManager">
            <bean class="com.yonyou.iuap.cache.CacheManager">
                <property name="jedisTemplate" ref="jedisTemplate"/>
            </bean>
        </property>
    </bean>
	</beans>
```

## 工程样例 ##

<img src="/articles/iuap-develop/9-/iuap-mdpersistence/3.1.0-RELEASE/images/mdjdbc_example.jpg"/>

## 常用接口 ##

### 基于元数据定义的单表数据的增删改查 ###

元数据持久化组件的增删改查逻辑to

```
	@Service
	public class UserService {

    @Autowired
    private MetadataDAO baseDAO;

    @Transactional(rollbackFor = DAOException.class)
    public String save(User user) throws DAOException {
        return baseDAO.insert(user);
    }

    @Transactional(rollbackFor = DAOException.class)
    public String update(User user) throws DAOException {
        return baseDAO.update(user);
    }

    @Transactional(rollbackFor = DAOException.class)
    public String remove(String id) throws DAOException {
        User user = new User();
        user.setId(id);
        return baseDAO.remove(user);
    }

    public String queryById(String id) throws DAOException {
        return baseDAO.queryByPK(User.class,id);
    }

     public Page queryPage(String name,PageRequest pageRequest) throws DAOException {
        SQLParameter parameter = new SQLParameter();
        parameter.addParam(name+"%");
        Page<IpuQuotation> page =
                baseDAO.queryPage("select * from users " + "where name like ?", parameter, pageRequest, IpuQuotation.class);
        return page;
    }
}
```


### 基于元数据定义的扩展表数据的增删改查功能  ###

如果元数据定义了数据的扩展，元数据持久化组件的api没有变化，针对于扩展属性，需要调用`setAttribute`方法设置值。

### 基于元数据定义的关联关系的查询功能 ###

关联查询功能使用`DASFacade`类来实现，类似代码如下:

```
     String[] paths =
                new String[] {"name", "count", "fk_employee.e_name", "fk_employee.e_code", "fk_bu.b_name",
                        "fk_bu.b_code", "fk_id_oderdt.name", "fk_id_oderdt.to_people", "fk_id_oderdt.from_people",
                        "fk_id_oderdt.ref_org.o_name", "fk_id_oderdt.ref_org.o_code"};
        SQLParameter sqlParameter = new SQLParameter();
        sqlParameter.addParam("1");
        sqlParameter.addParam("2");
        sqlParameter.addParam("3");

        List<ODerDt> dtList =
                metadataDAO.queryForList("select * from t_oddt where od_id in (?,?,?)", sqlParameter,
                        new BeanListMetaProcessor(ODerDt.class));
        Map<String, Map<String, Object>> resultList =
                DASFacade.getAttributeValueAsPKMap(paths, dtList.toArray(new ODerDt[] {}));
```

### 基于元数据操作的数据变更通知，默认提供日志记录 ###

元数据持久化组件对于元数据操作接口提供了数据变更通知功能，需要在`MetaDAO`的属性中设置`dataChangeNotifier`属性，组件默认提供了日志记录的功能，如果应用需要自定义扩展，需要实现`IDataChangeListener`接口，并做如下配置

```
 <bean id="dataChangeNotifier" class="com.yonyou.iuap.persistence.bs.notifier.DataChangeNotifier">
        <property name="dataChangeListenerList">
            <list>
                <bean class="xx.xxx.xx.CustomLister"/>
            </list>
        </property>
    </bean>
```


### 基于元数据定义，删除主表数据，级联删除子表数据的功能 ###

如果定义了关联关系，删除主表数据，持久化组件会自动删除所有子表数据。

### 基于元数据定义，删除数据时，如果数据被引用，不允许删除的功能 ###

如果定义了引用关系，删除数据时，如果该数据被其他数据引用，则不允许删除

### 基于元数据定义的应用数据缓存功能 ###

如果元数据定义了缓存属性，该元数据对应的应用数据会被缓存。

### 查询应用自定义缓存功能 ###

如果应用希望使用自己的缓存实现，元数据持久化组件提供了 `IMetaCache`接口，应用实现该接口完成自定义缓存的查询功能。

### 自定义接口功能 ###

元数据持久化组件支持应用数据自定义接口功能，使用`BizObject`类完成数据和自定义接口转换功能。







