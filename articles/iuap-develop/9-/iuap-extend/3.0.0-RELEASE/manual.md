#JPA持久化扩展组件#

##业务需求##
当业务在持久化使用上选择JPA框架时，因为Spring Data JPA的实现限制，有一些需求无法实现，比如所有操作统一进行事务控制，这在只读情况下会影响读取效率。

##解决方案##
本组件主要对Spring Data JPA和iuap-persistence组件中的JPA部分进行扩展，包括查询条件、数据库方言识别和事务的添加范围等功能。

如果业务上在使用默认的Spring Data JPA设置后没有扩展方面的需求，则可以不依赖此组件。

## 功能说明 ##
1.	扩展JPA的查询条件；
2.	扩展JPA的数据库方言识别；
3.	扩展JPA的事务添加范围，对只读操作不做事务控制；

# 使用说明 #

##配置Maven依赖##
**如果需要扩展JPA对事务的控制，在工程上依赖此组件，依赖方式如下：**

    根据项目的需求，排除冲突的版本，需要调整exclusions.
    <dependency>
        <groupId>com.yonyou.iuap</groupId>
        <artifactId>iuap-extend</artifactId>
        <version>${iuap.modules.version}</version>
        <exclusions>
            <exclusion>
                <artifactId>guava</artifactId>
                <groupId>com.google.guava</groupId>
            </exclusion>
            <exclusion>
                <artifactId>cglib</artifactId>
                <groupId>cglib</groupId>
            </exclusion>
        </exclusions>
    </dependency>

${iuap.modules.version}为在pom.xml中定义的需要引用的组件的版本。

##调整Spring配置文件##
**在对应的jpa方式的持久化相关的spring配置文件中，调整如下配置：**

    <!-- Spring Data Jpa配置 -->
    <jpa:repositories base-package="com.yonyou.iuap.repository"
                      transaction-manager-ref="transactionManager"
                      factory-class="org.springframework.data.jpa.repository.support.CustomJpaRepositoryFactoryBean"
                      entity-manager-factory-ref="entityManagerFactory"/>

	
**在服务调用需要控制事务的方法上，使用Spring的事务控制**

注意：Spring的持久化配置文件建议只有一份，统一规范数据源和事务的控制，尽量不要声明多个事务控制器。

##特殊扩展##

**特殊数据库和查询的扩展**

此组件中，对springside的查询和spring data jpa对数据库的兼容类进行扩展，如果需要spring data jpa扩展查询条件和特殊数据库兼容，请参考组件内部的方式

- 扩展对特殊数据库的支持
- 
	public class Hibernates {
		
		... ...
	
		/**
		 * 从DataSoure中取出connection, 根据connection的metadata中的jdbcUrl判断Dialect类型.
		 * 仅支持Oracle, H2, MySql, PostgreSql, SQLServer，如需更多数据库类型，请仿照此类自行编写。
		 */
		public static String getDialect(DataSource dataSource) {
			String jdbcUrl = getJdbcUrlFromDataSource(dataSource);
	
			// 根据jdbc url判断dialect
			if (StringUtils.contains(jdbcUrl, ":h2:")) {
				return H2Dialect.class.getName();
			} else if (StringUtils.contains(jdbcUrl, ":mysql:")) {
				return MySQL5InnoDBDialect.class.getName();
			} else if (StringUtils.contains(jdbcUrl, ":oracle:")) {
				return Oracle10gDialect.class.getName();
			} else if (StringUtils.contains(jdbcUrl, ":postgresql:")) {
				return PostgreSQL82Dialect.class.getName();
			} else if (StringUtils.contains(jdbcUrl, ":sqlserver:")) {
				return SQLServer2008Dialect.class.getName();
			} else if(StringUtils.contains(jdbcUrl, ":hsqldb:")) {
				return HSQLDialect.class.getName();
			} else {
				throw new IllegalArgumentException("Unknown Database of " + jdbcUrl);
			}
		}
	
		... ...
	}

- 扩展对特殊条件查询的支持

请参考修改DynamicSpecifications和SearchFilter类


其他详细配置和参数，请参考示例工程(DevTool/examples/example\_iuap\_persistence)和JPA相关文档