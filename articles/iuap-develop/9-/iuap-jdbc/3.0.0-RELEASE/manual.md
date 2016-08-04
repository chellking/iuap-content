# JDBC持久化组件 #

## 业务需求 ##
业务在面临持久化问题时，经常会遇到对不同数据源的匹配问题。在解决这一问题时，采用多套适配代码是一种常用手段，但是这种解决方式对开发影响较大，需要更多持久化工作，此外在持久化开发过程中还需要有一些常用的数据库建库建表操作，都需要平台的持久化组件统一屏蔽这类问题。

##解决方案##
iuap平台采用iuap-jdbc作为数据持久化组件，实现对业务数据的持久化服务。该组件对JDBC做了轻量封装，采用基于注解的ORM映射机制，使用简单，简化了代码配置，通过少量代码就可以完成持久化操作。组件能够自动适配多种数据源，提供不同数据库的基本语法转换能力，做到一套语法，多种数据库兼容执行，所有这些都对开发透明，开发者可以更关注于业务本身，屏蔽了底层数据库切换带来的影响。

## 功能说明 ##
1.	封装JDBC，基于注解的ORM映射机制，简化代码配置；
2.	面向对象编程，直接操作POJO，少量代码完成增删改查操作；
3.	自动识别适配数据源，提供多种数据库的语法转换，目前支持MySQL、Oracle和PostgreSQL;


# 功能特征 #

-  多数据库语法适配

目前组件适配了`mysql`,`oracle`和`postgresql`的通用语法。

- 数据源

多数据库持久化组件使用CrossdbDataSource装饰外部数据源，应用的数据源可以自由配置，组件在完成数据源的适配的同时，为多数据库的语法适配能力做铺垫。

- 自动分页

组件提供分页API，应用只需要传入查询条件sql，不必关心分页逻辑，组件自动帮您完成分页。

- 主子表组合保存，修改

组件提供主子表组合操作。

# 使用说明 #

## Maven依赖 ##
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-jdbc</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 相关配置 ##


## 简单示例 ##

使用iuap-jdbc完成单表的增删改查。示例包括实体类、DAO类、Service类。其中dao类通过注入BaseDao类，并使用它对数据进行操作。

**数据库表**

- user数据库表

		drop table if exists users;
		 
		create table users (
		  id char(36),
		  name varchar(20),
		  ts datetime,
		  dr int
		);

**spring配置文件**

-  在Spring中声明数据源和BaseDAO

    
	    <!-- 通用配置文件加载 -->
	    <context:property-placeholder system-properties-mode="OVERRIDE" ignore-unresolvable="true" location="classpath*:/application.properties"/>
	
	    <!-- 数据源配置, 使用Tomcat JDBC连接池 -->
	    <bean id="jdbcDataSource" class="org.apache.tomcat.jdbc.pool.DataSource" destroy-method="close">
	        <!-- Connection Info -->
	        <property name="driverClassName" value="${jdbc.driver}"/>
	        <property name="url" value="${jdbc.url}"/>
	        <property name="username" value="${jdbc.username}"/>
	        <property name="password" value="${jdbc.password}"/>
	        <property name="defaultCatalog" value="${jdbc.catalog}"/>
	
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
	    
	    <!-- iuap跨数据库数据库装饰类 -->
	    <bean id="crossDBDataSource" class="com.yonyou.iuap.persistence.bs.framework.ds.CrossdbDataSource">
	        <constructor-arg name="dataSource" ref="jdbcDataSource"/>
	    </bean>
	
	    <!-- spring 事务配置 -->
	    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	        <property name="dataSource" ref="crossDBDataSource"/>
	    </bean>
	
	    <!-- 使用annotation定义事务 -->
	    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
	
	    <!-- 声明spring-jdbcTemplate -->
	    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
	        <property name="dataSource" ref="crossDBDataSource"></property>
	    </bean>
	   
	    <!-- 声明BaseDAO -->
	    <bean id="baseDAO" class="com.yonyou.iuap.persistence.bs.dao.BaseDAO">
	        <property name="jdbcTemplate" ref="jdbcTemplate"/>
	        <property name="dbMetaHelper" ref="dbMetaInfo"/>
	    </bean>
	
	    <!-- 声明数据源辅助类 -->
	    <bean id="dbMetaInfo" class="com.yonyou.iuap.persistence.bs.util.DBMetaHelper">
	        <property name="jdbcTemplate" ref="jdbcTemplate"/>
	    </bean>

**实体类**

- 生成User.java对象

   		@Entity(name = "User",namespace="your.package.name")
    	@Table(name = "users")
    	public class User extends BaseEntity{
    	
    		@Id
    		@Column(name = "id")
    		@GeneratedValue(strategy = Stragegy.UUID, moudle = "users")
    		private String id;
    		
    		@Column(name = "name")
    		private String name;
    		
    		@Column(name="ts")
    		private Date ts;
    		   
    		@Column(name="dr")
    		private Integer dr;
    		
    		//省略getter,setter...
    		
    		public String getMetaDefinedName() {
    			return "User";
    		}
    		
    		public String getNamespace() {
    			return "your.package.name";
    		}
    	}

注意:

实体类需要继承com.yonyou.iuap.persistence.vo.BaseEntity类。

@Id，@coloumn等注解是iuap-jdbc中的注解，请开发者注意和javax包下的注解进行区分，不要混淆。 @GeneratedValue(strategy = Stragegy.UUID, moudle = "users")注解中的ID生成策略可以参考Stragegy中的枚举值，和iuap-oid中的生成策略保持一致。

getMetaDefinedName和getNamespace方法是为了后期的元数据操作时候预留，如果不需要元数据，开发人员手动重写即可（必须有该方法）。

业务代码中不需要操作操作`User`对象的`ts`属性，但是每次修改操作ts属性都会变成服务器的当前时间。组件的**sql增强功能**会自动在增加或修改的时候，在`sql`上增加`ts`字段，使得应用更加专注于业务相关的逻辑。

**DAO类**

- 推荐新建一个实体类User的DAO

        @Repository
    	public class UserDao {
    	
    		//BaseDAO是iuap-jdbc的基础dao类，该类中包含了一些通用的数据库操作
    		@Autowired
   			private BaseDAO dao;
    	
    		public User queryByID(String id) throws DAOException {
    			return dao.queryByPK(User.class, id);
    		}
    		
    		public void save(User user) throws DAOException {
    			dao.insert(user);
    		}
    		
    		public void remove(User user) throws DAOException {
    			dao.remove(user);
    		}
    	
    		public void remove(List<User> users) throws DAOException {
    			dao.remove(users);
    		}
    		
    		public String update(User user) throws DAOException {
    			return dao.update(user);
    		}
    	
    	
    		public Page queryPage(String name,PageRequest pageRequest) throws 	DAOException {
       			SQLParameter parameter = new SQLParameter();
       			parameter.addParam(name+"%");
       			Page<User> page =dao.queryPage("select * from users " + "where name like ?", parameter, pageRequest, User.class);
       			return page;
    		}
    	}

组件提供分页API，应用只需要传入查询条件sql，不必关心分页逻辑，组件自动帮您完成分页。详见`BaseDAO`的API `queryPage()`

**Service类**

- 创建实体类User的Service


		@Service
		public class UserService {
	
		    @Autowired
		    private UserDao userDao;
		
		    @Transactional(rollbackFor = DAOException.class)
		    public String save(User user) throws DAOException {
		        return userDao.save(user);
		    }
		
		    @Transactional(rollbackFor = DAOException.class)
		    public String update(User user) throws DAOException {
		        return userDao.update(user);
		    }
		    
		    @Transactional(rollbackFor = DAOException.class)
		    public String remove(String id) throws DAOException {
		        User user = new User();
		        user.setId(id);
		        return userDao.remove(user);
		    }
		    
		    public String queryById(String id) throws DAOException {
		        return userDao.queryByPK(User.class,id);
		    }
		    
		     public Page queryPage(String name,PageRequest pageRequest) throws DAOException {
		        return userDao.queryPage(name,pageRequest);
	   		}
		}


##主子表示例##

**主表实体**

    @Entity
    @Table(name = "iuap_parent")
    public class Parent extends BaseEntity{
    
    	private static final long serialVersionUID = -4725617307982134263L;
    
    	@Id
    	@Column(name = "id")
    	private String id;
    	
    	@Column(name = "id")
    	private String name;	
    	
    	@Column(name = "ts")
    	private java.util.Date ts;  	
    	
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
    	public Date getTs() {
    		return ts;
    	}
    	public void setTs(Date ts) {
    		this.ts = ts;
    	}
    	  	
    	@Override
    	public String getMetaDefinedName() {
    		return "iuap_parent";
    	}
    
   		@Override
    	public String getNamespace() {
    		return "iuap_parent";
    	}
    	
    }

**子表实体**
    
    @Entity
    @Table(name = "iuap_child")
    public class Child extends BaseEntity{
    
    	/**
    	 * 
    	 */
    	private static final long serialVersionUID = -6415805183878684588L;
    	
    	@Id
    	@Column(name = "id")
    	private String id;
    	
    	//通过外键注解与主表建立关联referenceTableName为主表表名，referencedColumnName为主表主键
    	@FK(referenceTableName="iuap_parent",referencedColumnName="id", name = "")  
    	@Column(name = "parentid")
    	private String parentid;
    	
    	@Column(name = "name")
    	private String name;	
    	
    	@Column(name = "ts")
    	private java.util.Date ts;
    
    	public String getId() {
    		return id;
    	}
    
    	public void setId(String id) {
    		this.id = id;
    	}
    
    	public String getParentid() {
    		return parentid;
    	}
    
    	public void setParentid(String parentid) {
    		this.parentid = parentid;
    	}
    
    	public String getName() {
    		return name;
    	}
    
    	public void setName(String name) {
    		this.name = name;
    	}
    
    	public java.util.Date getTs() {
    		return ts;
    	}
    
    	public void setTs(java.util.Date ts) {
    		this.ts = ts;
    	}
    
    	@Override
    	public String getMetaDefinedName() {
    		return "iuap_child";
    	}
    
    	@Override
    	public String getNamespace() {
    		return "iuap_child";
    	}
    }



**主子联合实体**
    
    public class CombineVO {
    	
    	private Parent parent;
    	
    	private List<Child> childs;
    
    	public Parent getParent() {
    		return parent;
    	}
    
    	public void setParent(Parent order) {
    		this.parent = order;
    	}
    
    	public List<Child> getChilds() {
    		return childs;
    	}
    
    	public void setChilds(List<Child> childs) {
    		this.childs = childs;
    	}
    	
    	
    
    }
        

**主表dao**
    
    public class ParentDao {
    	
    	@Autowired
    	private BaseDAO dao;
    	
    	
    	public Parent queryByPK(String pk) throws DAOException {
    		
    		return dao.queryByPK(Parent.class, pk);
    	}
    	
    	public Page<Parent> queryPage(Map<String, Object> searchParams, PageRequest pageRequest) throws DAOException {
    		StringBuffer sqlBuffer = new StringBuffer("select * from iuap_parent where 1=1 ");
    		SQLParameter sqlParameter = new SQLParameter();
    		buildSql(searchParams, sqlBuffer, sqlParameter);
    		String sql = sqlBuffer.toString();
    		return dao.queryPage(sql, sqlParameter, pageRequest, Parent.class);
    	}
    
    	public void save(Parent vo) throws DAOException {
    		dao.save(vo);
    	}
    	
    	public void save(Parent vo, Child[] details) throws DAOException {
    		dao.save(vo, details);
    	}
    
    	public void remove(Parent vo) throws DAOException {
    		dao.remove(vo);
    	}
    
    	public void remove(List<Parent> vos) throws DAOException {
    		dao.remove(vos);
    	}
    
    	//业务开发根据自己的需求，修改查询条件的拼接方式
    	private void buildSql(Map<String, Object> searchParams, StringBuffer sqlBuffer, SQLParameter sqlParameter) {
    		
    		int index = 0;
    		StringBuffer sb = new StringBuffer();
    		for (Map.Entry<String, Object> entry : searchParams.entrySet()) {
    			String[] keySplit = entry.getKey().split("__");//xzn
    			if (keySplit.length == 2) {
    				String columnName = keySplit[1];
    				String compartor = keySplit[0];
    				Object value = entry.getValue();
    				if (value != null && StringUtils.isNotBlank(value.toString())) {
    					
    					sb.append(columnName).append(" ").append(compartor).append(" ? ");
    					// 处理模糊查询
    					value = "like".equalsIgnoreCase(compartor) ? value + "%" : value;
    					sqlParameter.addParam(value);
    					index ++;
    					
    					if(index != searchParams.keySet().size()){
    						sb.append(" or ");
    					}
    				}
    			}
    		}
    		
    		String conditionSql = sb.toString();
    		if(StringUtils.isNoneBlank(conditionSql)){
    			sqlBuffer.append(" and (" + conditionSql.toString() + ");");
    		}
    		
    	}
    
    }
    
**子表dao**
    
    public class ChildDao {
    	@Autowired
    	private BaseDAO dao;
    	
    	public Child queryByPK(String pk) throws DAOException {
    		return dao.queryByPK(Child.class, pk);
    	}
    
    	public List<Child> queryChildByParentId(String parentId) throws DAOException {
    		String sql = "select * from iuap_child where parentid = ? ";
    		SQLParameter sqlParameter = new SQLParameter();
    		sqlParameter.addParam(parentId);
    		return dao.queryByClause(Child.class, sql,sqlParameter);
     	}
    	
    	public void save(Child vo) throws DAOException {
    		dao.save(vo);
    	}
    	public void update(List<Child>  vo) throws DAOException {
    		dao.updateOptional(vo);
    	}
    
    	public void remove(Child vo) throws DAOException {
    		dao.remove(vo);
    	}
    	
    	public void remove(List<Child> vos) throws DAOException {
    		dao.remove(vos);
    	}
    	
    	//业务开发根据自己的需求，修改查询条件的拼接方式
    	private void buildSql(Map<String, Object> searchParams, StringBuffer sqlBuffer, SQLParameter sqlParameter) {
    		
    		int index = 0;
    		StringBuffer sb = new StringBuffer();
    		for (Map.Entry<String, Object> entry : searchParams.entrySet()) {
    			String[] keySplit = entry.getKey().split("__");//xzn
    			if (keySplit.length == 2) {
    				String columnName = keySplit[1];
    				String compartor = keySplit[0];
    				Object value = entry.getValue();
    				if (value != null && StringUtils.isNotBlank(value.toString())) {
    					
    					sb.append(columnName).append(" ").append(compartor).append(" ? ");
    					// 处理模糊查询
    					value = "like".equalsIgnoreCase(compartor) ? value + "%" : value;
    					sqlParameter.addParam(value);
    					index ++;
    					
    					if(index != searchParams.keySet().size()){
    						sb.append(" or ");
    					}
    				}
    			}
    		}
    		
    		String conditionSql = sb.toString();
    		if(StringUtils.isNoneBlank(conditionSql)){
    			sqlBuffer.append(" and (" + conditionSql.toString() + ");");
    		}
    		
    	}
    
    }
    
**主表service**
    
    @Service
    public class ParentService {
    	
    	@Autowired
    	private ParentDao dao;
    	
    	@Autowired
    	private ChildDao childDao;
    	
    
    	public Parent getTestorderById(String id) throws DAOException {
    		return dao.queryByPK(id);
    	}
    
    	@Transactional
    	public void deleteById(String id) throws DAOException {
    		Parent parent = new Parent();
    		parent.setId(id);
    		dao.remove(parent);
    	}
    	
    	@Transactional
    	public void batchDelete(List<String> ids) throws DAOException {
    		List<Parent> deleteVos = new ArrayList<Parent>();
    		for (int i = 0; i < ids.size(); i++) {
    			Parent parent = new Parent();
    			parent.setId(ids.get(i));
    			deleteVos.add(parent);
    		}
    		if (deleteVos.size() > 0) {
    			dao.remove(deleteVos);
    		}
    	}
    	
    	@Transactional
    	public Parent saveEntity(Parent entity) throws DAOException {
    		dao.save(entity);
    		return entity;
    	}
    	
    	@Transactional
    	public void saveAll(CombineVO vo) throws DAOException {
    		Parent parent = vo.getParent();
    		if(parent.getId()!=null){
    			parent.setStatus(VOStatus.UPDATED);
    		} else {
    			parent.setStatus(VOStatus.NEW);
    		}
    		
    		List<Child> childs = vo.getChilds();
    		for (int i = 0; i < childs.size(); i++) {
    			Child Child = childs.get(i);
    			if(StringUtils.isBlank(Child.getId())){
    				Child.setStatus(VOStatus.NEW);
    			} else {
    				Child.setStatus(VOStatus.UPDATED);
    			}
    		}
    		Child[] children = childs.toArray(new Child[0]);
    		
    		dao.save(parent, children);
    	}
    
    	public Page<Parent> getPage(Map<String, Object> searchParams, PageRequest pageRequest) throws DAOException {
    		return dao.queryPage(searchParams, pageRequest);
    	}
    
    
    }
    

**子表service**

    
    @Service
    public class ChildService {
    	
    	@Autowired
    	private ChildDao dao;
    	
    	public Child getTestorderDetailById(String id) throws DAOException {
    		return dao.queryByPK(id);
    	}
    
    	@Transactional
    	public void deleteById(String id) throws DAOException {
    		Child child = new Child();
    		child.setId(id);
    		dao.remove(child);
    	}
    	
    	@Transactional
    	public void batchDelete(List<String> ids) throws DAOException {
    		List<Child> deleteVos = new ArrayList<Child>();
    		for (int i = 0; i < ids.size(); i++) {
    			Child child = new Child();
    			child.setId(ids.get(i));
    			deleteVos.add(child);
    		}
    		if (deleteVos.size() > 0) {
    			dao.remove(deleteVos);
    		}
    	}
    	
    	@Transactional
    	public Child saveEntity(Child entity) throws DAOException {
    		dao.save(entity);
    		return entity;
    	}
    
    	public List<Child> queryListByParentId(String parentId) throws DAOException {
    		return dao.queryChildByParentId(parentId);
    	}
    
    }

组件提供主子表组合操作。API对应`BaseDAO`的`save()`,入参为一主多子。
**注意**
使用这个方法的时候，Entity的status属性必须设置。

**status说明如下：**
<table style="border-collapse:collapse">
	<tr><th>status值</th><th>说明</th></tr>
    <tr><td>0</td><td>本条记录未变更</td></tr>
	<tr><td>1</td><td>本条记录被修改</td></tr>
	<tr><td>2</td><td>本条记录为新增数据</td></tr>
	<tr><td>3</td><td>本条记录被删除(主表数据状态不能为删除状态)</td></tr>
</table>


#iuap-jdbc相关API#

- BaseDAO

<table style="border-collapse:collapse">
	<tr>
		<th>方法名</th>
		<th>参数</th>
		<th>返回值</th>
		<th>功能说明</th>
	</tr>

	<tr>
		<td>queryByPK</td>
		<td>Class&lt;T&gt; className（查询实体类类型）, ID pk（需要查询的实体类的主键键值）</td>
		<td> T（查询到的实体类）</td>
		<td>通过主键查询实体</td>
	</tr>

	<tr>
		<td>queryByPK</td>
		<td>Class&lt;T&gt; className（查询实体类类型）, ID pk（需要查询的实体类的主键键值），String[] selectedFields（指定实体的返回列）</td>
		<td> T（查询到的实体类）</td>
		<td>通过主键查询实体（指定实体的返回列）</td>
	</tr>

	<tr>
		<td>queryByClause</td>
		<td>Class&lt;T&gt; className（查询实体类类型）,  String sql（查询的sql语句）</td>
		<td>List&lt;T&gt;（查询到的实体列表）</td>
		<td>通过sql语句查询实体</td>
	</tr>

	<tr>
		<td>queryByClause</td>
		<td>Class&lt;T&gt; className（查询实体类类型）,  String sql（查询的sql语句），SQLParameter parameter（sql语句拼接的参数）</td>
		<td>List&lt;T&gt;（查询到的实体列表）</td>
		<td>通过sql语句查询实体</td>
	</tr>

	<tr>
		<td>queryAll</td>
		<td>Class&lt;T&gt; className（查询实体类类型）</td>
		<td>List&lt;T&gt;（查询到的实体列表）</td>
		<td>查询该实体的所有记录</td>
	</tr>

	<tr>
		<td>queryForList</td>
		<td>String sql(查询sql语句), SQLParameter parameter（sql参数）, ResultSetProcessor processor(查询结果解析器)</td>
		<td>List&lt;T&gt;（返回结果列表）</td>
		<td>根据查询结果解析器返回相应的查询结果列表</td>
	</tr>

	<tr>
		<td>queryForList</td>
		<td>String sql(查询sql语句),ResultSetProcessor processor(查询结果解析器)</td>
		<td>List&lt;T&gt;（返回结果列表）</td>
		<td>根据查询结果解析器返回相应的查询结果列表</td>
	</tr>

	<tr>
		<td>queryForObject</td>
		<td>String sql(查询sql语句), SQLParameter parameter（sql参数）,ResultSetProcessor processor(查询结果解析器)</td>
		<td>T（返回结果）</td>
		<td>根据查询结果解析器返回相应的查询结果</td>
	</tr>
	
	<tr>
		<td>queryForObject</td>
		<td>String sql(查询sql语句),ResultSetProcessor processor(查询结果解析器)</td>
		<td>T（返回结果）</td>
		<td>根据查询结果解析器返回相应的查询结果</td>
	</tr>

	<tr>
		<td>queryPage</td>
		<td>String sql(查询sql语句), SQLParameter parameter(查询sql参数), PageRequest pageRequest(分页请求参数), Class&lt;T&gt; type（查询实体类类型）</td>
		<td>Page&lt;T&gt;（返回结果分页）</td>
		<td>分页查询</td>
	</tr>

	<tr>
		<td>insert</td>
		<td>T t(待插入的实体)</td>
		<td>ID(插入实体类的主键键值)</td>
		<td>插入新实体</td>
	</tr>

	<tr>
		<td>insert</td>
		<td>List&lt;T&gt; vos(待插入的实体列表)</td>
		<td>ID[](插入实体类的主键键值列表)</td>
		<td>插入多个新实体</td>
	</tr>

	<tr>
		<td>insertWithPK</td>
		<td>T vo(待插入的实体)</td>
		<td>ID(插入实体类的主键键值)</td>
		<td>插入新实体（自带主键键值）</td>
	</tr>
	
	<tr>
		<td>insertWithPK</td>
		<td>List&lt;T&gt; vos(待插入的实体列表)</td>
		<td>ID[](插入实体类的主键键值列表)</td>
		<td>插入多个新实体（自带主键键值）</td>
	</tr>

	<tr>
		<td>insertOptional</td>
		<td>BaseEntity vo（实体）</td>
		<td>ID（实体主键）</td>
		<td>存储实体数据（选择性保存字段）</td>
	</tr>

	<tr>
		<td>insertOptional</td>
		<td>List<? extends BaseEntity> list（实体列表）</td>
		<td>ID[]（实体主键列表）</td>
		<td>存储实体数据（选择性保存字段）</td>
	</tr>

	<tr>
		<td>insertOptionalWithPK</td>
		<td>BaseEntity vo（实体）</td>
		<td>ID（实体主键）</td>
		<td>存储实体数据自带主键键值（选择性保存字段）</td>
	</tr>

	<tr>
		<td>insertOptionalWithPK</td>
		<td>List<? extends BaseEntity> list（实体列表）</td>
		<td>ID[]（实体主键列表）</td>
		<td>存储实体数据自带主键键值（选择性保存字段）</td>
	</tr>

	<tr>
		<td>update</td>
		<td>T vo(更新实体)</td>
		<td>int (更新的行数)</td>
		<td>更新数据</td>
	</tr>

	<tr>
		<td>update</td>
		<td>List&lt;T&gt; vos(更新实体列表)</td>
		<td>int (更新的行数)</td>
		<td>更新数据</td>
	</tr>

	<tr>
		<td>update</td>
		<td>T vo(更新实体)，String... fieldNames（需要更新的实体字段）</td>
		<td>int (更新的行数)</td>
		<td>更新数据的特定字段</td>
	</tr>

	<tr>
		<td>update</td>
		<td>List&lt;T&gt; vos(更新实体列表)，String... fieldNames（需要更新的实体字段）</td>
		<td>int (更新的行数)</td>
		<td>更新多个数据的特定字段</td>
	</tr>

	<tr>
		<td>update</td>
		<td>String sql（sql语句）</td>
		<td>int (更新的行数)</td>
		<td>通过sql进行更新</td>
	</tr>

	<tr>
		<td>update</td>
		<td>String sql（sql语句）, SQLParameter parameter（sql函数）</td>
		<td>int (更新的行数)</td>
		<td>通过sql进行更新</td>
	</tr>

	<tr>
		<td>updateOptional</td>
		<td>BaseEntity vo(待更新实体类)</td>
		<td>int (更新的行数)</td>
		<td>更新实体数据（选择性保存字段）</td>
	</tr>
	
	<tr>
		<td>updateOptional</td>
		<td>List<? extends BaseEntity> list（实体列表）</td>
		<td>int (更新的行数)</td>
		<td>更新多个实体数据（选择性保存字段）</td>
	</tr>

	<tr>
		<td>remove</td>
		<td>T vo(待删除的实体)</td>
		<td>void</td>
		<td>删除数据</td>
	</tr>
	
	<tr>
		<td>remove</td>
		<td>List&lt;T&gt; vos(待删除的实体列表)</td>
		<td>void</td>
		<td>批量删除数据</td>
	</tr>

	<tr>
		<td>save</td>
		<td>BaseEntity parent(主实体), BaseEntity... children（子实体）</td>
		<td>void</td>
		<td>存储主子实体</td>
	</tr>

</table>

  
## 其他相关说明 ##

### 查询结果解析器 ###

组件提供了多种查询结果解析器。解析器类名以及说明如下：

<table style="border-collapse:collapse">
	<tr><th>解析器类名</th><th>说明</th></tr>
	<tr><td>BeanProcessor</td><td>值对象集合处理器，返回一个ArrayList集合，集合中的每一个元素是一个javaBean,每个javaBean对应结果集合中一行数据</td></tr>
	<tr><td>BeanListProcessor</td><td>值对象集合处理器，返回一个ArrayList集合，集合中的每一个元素是一个javaBean,每个javaBean对应结果集合中一行数据</td></tr>
	<tr><td>ColumnProcessor</td><td>列值处理器，返回一个Java对象，结果集中只有一行数据，该对象对应与结果集中某一列的值，该处理器通过结果集列的序号或名称来确定列</td></tr>
	<tr><td>ColumnListProcessor</td><td>列值处理器，返回一个ArrayList对象，结果集中有多行数据，该对象对应与结果集中某一列的值，该处理器通过结果集列的序号或名称来确定列</td></tr>
	<tr><td>MapProcessor</td><td>HashMap处理器，返回一个HashMap, 结果集中只有一行数据，其中结果集合中每一列的列名和列值对应HashMap的一个关键字和相应的值</td></tr>
    <tr><td>MapListProcessor</td><td>HashMap集合处理器，返回一个ArrayList集合，集合中的每一个元素是一个HashMap，每个HashMap对应结果集中的一行数据,其中结果集合中每一列的列名和列值对应HashMap的一个关键字和相应的值</td></tr>

    <tr><td>ArrayProcessor</td><td>数组处理器，返回一个对象数组，结果集中只有一行数据，其中结果集中每一列对应数组的一个元素</td></tr>
    <tr><td>ArrayListProcessor</td><td>数组集合处理器，返回一个ArrayList集合，集合中的每一个元素是一个数组，每个数组对应结果集中的一行数据，其中结果集中每一列对应数组的一个元素</td></tr>
</table>


### 函数转换对应表 ###

- `mysql`

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>iuap-jdbc函数</th>
			<th>mysql函数</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>PI()</td>
			<td>3.1415926535897931</td>
		</tr>
		<tr>
			<td>%(模运算) a % b</td>
			<td>mod(a,b)</td>
		</tr>
		<tr>
			<td>getdate()</td>
			<td>to_char(now(),'yyyy-mm-dd hh24:mi:ss')</td>
		</tr>
		<tr>
			<td>square</td>
			<td>power(num,2)</td>
		</tr>
		<tr>
			<td>&amp;&amp; (a&amp;&amp;b&amp;&amp;c)</td>
			<td>, (a,b,c)</td>
		</tr>
		<tr>
			<td>top n</td>
			<td>select * from table limit n</td>
		</tr>
		<tr>
			<td>patindex('pattern',column)</td>
			<td>instr('pattern',column)</td>
		</tr>
		<tr>
			<td>dateadd(dateType,num,date)</td>
			<td>参数为类型(year,month,day)，数值，时间</td>
		</tr>
		<tr>
			<td>dateadd(yy/yyyy/year,num,date)</td>
			<td>DATE_ADD( date_format(date,'%Y-%m-%d') , INTERVAL num YEAR)</td>
		</tr>
		<tr>
			<td>dateadd(mm/m/month,num,date)</td>
			<td>DATE_ADD( date_format('date','%Y-%m-%d') , INTERVAL num MONTH)</td>
		</tr>
		<tr>
			<td>dateadd(other,num,date)</td>
			<td>DATE_ADD( date_format(date,'%Y-%m-%d') , INTERVAL num DAY)</td>
		</tr>
		<tr>
			<td>datediff(dateType,start,end)</td>
			<td>参数类型(同上)，开始，结束</td>
		</tr>
		<tr>
			<td>datediff(year/yyyy/yy,start,end)</td>
			<td>PERIOD_DIFF(date_format(start,'%Y'),date_format(end,'%Y'))</td>
		</tr>
		<tr>
			<td>datediff(m/mm/month,start,end)</td>
			<td>PERIOD_DIFF(date_format(start,'%Y%m'),date_format(end,'%Y%m'))</td>
		</tr>
		<tr>
			<td>datediff(year/yyyy/yy,start,end)</td>
			<td>PERIOD_DIFF(date_format(start,'%Y%m%d'),date_format(end,'%Y%m%d'))</td>
		</tr>
		<tr>
			<td>case when then else end</td>
			<td>语法支持，未处理</td>
		</tr>
		<tr>
			<td>select column as b from table as alias</td>
			<td>select column b from table alias</td>
		</tr>
		<tr>
			<td>isnull</td>
			<td>ifnull</td>
		</tr>
		<tr>
			<td>len</td>
			<td>length</td>
		</tr>
		<tr>
			<td>count ( * )</td>
			<td>count(*)</td>
		</tr>
		<tr>
			<td>ltrim_case</td>
			<td>ltrim</td>
		</tr>
		<tr>
			<td>rtrim_case</td>
			<td>rtrim</td>
		</tr>
		<tr>
			<td>left(str,n)</td>
			<td>语法支持，未处理</td>
		</tr>
		<tr>
			<td>right(str,n)</td>
			<td>语法支持，未处理</td>
		</tr>
		<tr>
			<td>coalesce(xx,xxx...,notnull)</td>
			<td>语法支持，未处理</td>
		</tr>
		<tr>
			<td>round(str)</td>
			<td>语法支持，未处理</td>
		</tr>
		<tr>
			<td>convert(日期类型,xxx)</td>
			<td>语法支持，未处理</td>
		</tr>
		<tr>
			<td>cast</td>
			<td>语法支持，未处理</td>
		</tr>
	</tbody>
</table>

- `oracle`

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>iuap-jdbc函数</th>
			<th>oracle函数</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>PI()</td>
			<td>3.1415926535897931</td>
		</tr>
		<tr>
			<td>%(模运算) a % b</td>
			<td>mod(a,b)</td>
		</tr>
		<tr>
			<td>getdate()</td>
			<td>sysdate</td>
		</tr>
		<tr>
			<td>&amp;&amp; (a&amp;&amp;b&amp;&amp;c)</td>
			<td>, (a,b,c)</td>
		</tr>
		<tr>
			<td>top n</td>
			<td>where rownum&lt;n+1</td>
		</tr>
		<tr>
			<td>case when then else end</td>
			<td>decode()</td>
		</tr>
		<tr>
			<td>select column as b from table as alias</td>
			<td>select column b from table alias</td>
		</tr>
		<tr>
			<td>left(str,n)</td>
			<td>substr(str,1,n)</td>
		</tr>
		<tr>
			<td>right(str,n)</td>
			<td>substr(str,length(str)-n+1,n)</td>
		</tr>
		<tr>
			<td>coalesce(xx,xxx...,notnull)</td>
			<td>nvl()嵌套 nvl(nvl(1,2),3)</td>
		</tr>
		<tr>
			<td>square</td>
			<td>power(num,2)</td>
		</tr>
		<tr>
			<td>patindex('pattern',column)</td>
			<td>instr ( column, 'pattern', 1, 1 )</td>
		</tr>
		<tr>
			<td>len(str)</td>
			<td>length(rtrim(str,' '))</td>
		</tr>
		<tr>
			<td>ltrim(str)</td>
			<td>ltrim(str,' ')</td>
		</tr>
		<tr>
			<td>rtrim(str)</td>
			<td>rtrim(str,' ')</td>
		</tr>
		<tr>
			<td>round(str)</td>
			<td>rtrim(str,' ')</td>
		</tr>
		<tr>
			<td>cast(日期类型,xxx)</td>
			<td>to_date(xxx,\'yyyy-mm-dd hh24:mi:ss')</td>
		</tr>
		<tr>
			<td>cast(char(len),xxx)</td>
			<td>substr(to_char(xxx),1,len)</td>
		</tr>
		<tr>
			<td>cast(char,xxx)</td>
			<td>to_char(xxx)</td>
		</tr>
		<tr>
			<td>cast(其他类型,xxx)</td>
			<td>cast(xxx as 其他类型)</td>
		</tr>
		<tr>
			<td>convert()</td>
			<td>同cast</td>
		</tr>
		<tr>
			<td>getdate()</td>
			<td>sysdate</td>
		</tr>
		<tr>
			<td>dateadd(dateType,num,date)</td>
			<td>参数为类型(year,month,day)，数值，时间</td>
		</tr>
		<tr>
			<td>dateadd中的date</td>
			<td>to_date(substr(date,1,10),'yyyy-mm-dd')</td>
		</tr>
		<tr>
			<td>dateadd(yy/yyyy/year,num,date)</td>
			<td>add_months( to_date(substr(date,1,10),'yyyy-mm-dd'), (num*12))</td>
		</tr>
		<tr>
			<td>dateadd(mm/m/month,num,date)</td>
			<td>add_months( to_date(substr(date,1,10),'yyyy-mm-dd'), num)</td>
		</tr>
		<tr>
			<td>dateadd(other,num,date)</td>
			<td>to_date(substr(date,1,10),'yyyy-mm-dd')+num</td>
		</tr>
		<tr>
			<td>datediff(dateType,start,end)</td>
			<td>参数类型(同上)，开始，结束</td>
		</tr>
		<tr>
			<td>datediff(year/yyyy/yy,start,end)</td>
			<td>extract(year from to_date(substr(end,1,10),'yyyy-mm-dd')) - extract(year from to_date(substr(start,1,10),'yyyy-mm-dd'))</td>
		</tr>
		<tr>
			<td>datediff(m/mm/month,start,end)</td>
			<td>months_between( to_date(substr(end,1,10),'yyyy-mm-dd'), to_date(substr(start,1,10),'yyyy-mm-dd'))</td>
		</tr>
		<tr>
			<td>datediff(year/yyyy/yy,start,end)</td>
			<td>(to_date(substr(end,1,10),'yyyy-mm-dd')-to_date(substr(start,1,10),'yyyy-mm-dd'))</td>
		</tr>
		<tr>
			<td>substring</td>
			<td>substring(xxx,start,end)</td>
		</tr>
		<tr>
			<td>substring</td>
			<td>substr</td>
		</tr>
		<tr>
			<td>isnull</td>
			<td>nvl</td>
		</tr>
		<tr>
			<td>ceiling</td>
			<td>ceil</td>
		</tr>
		<tr>
			<td>ltrim_case</td>
			<td>ltrim</td>
		</tr>
		<tr>
			<td>rtrim_case</td>
			<td>rtrim</td>
		</tr>
	</tbody>
</table>

- `postgresql`

<table style="border-collapse:collapse">
	<thead>
		<tr>
			<th>iuap-jdbc函数</th>
			<th>pg函数</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>PI()</td>
			<td>3.1415926535897931</td>
		</tr>
		<tr>
			<td>%(模运算) a % b</td>
			<td>mod(a,b)</td>
		</tr>
		<tr>
			<td>getdate()</td>
			<td>to_char(now(),'yyyy-mm-dd hh24:mi:ss')</td>
		</tr>
		<tr>
			<td>&amp;&amp; (a&amp;&amp;b&amp;&amp;c)</td>
			<td>, (a,b,c)</td>
		</tr>
		<tr>
			<td>top n</td>
			<td>select * from table limit n</td>
		</tr>
		<tr>
			<td>case when then else end</td>
			<td>decode()</td>
		</tr>
		<tr>
			<td>select column as b from table as alias</td>
			<td>select column b from table alias</td>
		</tr>
		<tr>
			<td>left(str,n)</td>
			<td>substring(str,1,n)</td>
		</tr>
		<tr>
			<td>right(str,n)</td>
			<td>substring(str,length(str)-n+1,n)</td>
		</tr>
		<tr>
			<td>coalesce(xx,xxx...,notnull)</td>
			<td>nvl()嵌套 nvl(nvl(1,2),3)</td>
		</tr>
		<tr>
			<td>square</td>
			<td>power(num,2)</td>
		</tr>
		<tr>
			<td>patindex('pattern',column)</td>
			<td>position ( 'pattern' in clumn )</td>
		</tr>
		<tr>
			<td>len(str)</td>
			<td>length(rtrim(str,' '))</td>
		</tr>
		<tr>
			<td>round(str)</td>
			<td>rtrim(str,' ')</td>
		</tr>
		<tr>
			<td>convert(日期类型,xxx)</td>
			<td>to_date(xxx,'yyyy-mm-dd')</td>
		</tr>
		<tr>
			<td>convert(char(len),xxx)</td>
			<td>substring(to_char(xxx),1,len)</td>
		</tr>
		<tr>
			<td>convert(char,xxx)</td>
			<td>to_char(xxx)</td>
		</tr>
		<tr>
			<td>convert(其他类型,xxx)</td>
			<td>cast(xxx as 其他类型)</td>
		</tr>
		<tr>
			<td>getdate()</td>
			<td>to_char(now(),\'yyyy-mm-dd hh24:mi:ss')</td>
		</tr>
		<tr>
			<td>dateadd(dateType,num,date)</td>
			<td>参数为类型(year,month,day)，数值，时间</td>
		</tr>
		<tr>
			<td>dateadd(yy/yyyy/year,num,date)</td>
			<td>( to_date(to_char(date,\'yyyy-mm-dd'), 'yyyy-mm-dd') + interval 'n years')</td>
		</tr>
		<tr>
			<td>dateadd(mm/m/month,num,date)</td>
			<td>( to_date(to_char(date,'yyyy-mm-dd'), 'yyyy-mm-dd') + interval 'n months')</td>
		</tr>
		<tr>
			<td>dateadd(other,num,date)</td>
			<td>( to_date(to_char(date,'yyyy-mm-dd'), 'yyyy-mm-dd') + interval 'n days')</td>
		</tr>
		<tr>
			<td>datediff(dateType,start,end)</td>
			<td>参数类型(同上)，开始，结束</td>
		</tr>
		<tr>
			<td>datediff(year/yyyy/yy,start,end)</td>
			<td>extract(year from to_date(substring(end,1,10), 'yyyy-mm-dd')) - extract(year from to_date(substring(start,1,10), 'yyyy-mm-dd'))</td>
		</tr>
		<tr>
			<td>datediff(m/mm/month,start,end)</td>
			<td>extract(year from age(to_date(substring(end,1,10), 'yyyy-mm-dd'),to_date(substring(start,1,10), 'yyyy-mm-dd'))) * 12 + extract(month from age(to_date(substring(end,1,10), 'yyyy-mm-dd'),to_date(substring(start,1,10), 'yyyy-mm-dd')))</td>
		</tr>
		<tr>
			<td>datediff(year/yyyy/yy,start,end)</td>
			<td>(to_date(substring(end,1,10), 'yyyy-mm-dd')-to_date(substring(start,1,10), 'yyyy-mm-dd'))</td>
		</tr>
		<tr>
			<td>isnull</td>
			<td>coalesce</td>
		</tr>
		<tr>
			<td>len</td>
			<td>length</td>
		</tr>
		<tr>
			<td>ltrim_case</td>
			<td>ltrim</td>
		</tr>
		<tr>
			<td>rtrim_case</td>
			<td>rtrim</td>
		</tr>
	</tbody>
</table>

**更多详细的使用方法，请参考示例工程(DevTool/examples/example-iuap-jdbc)**

