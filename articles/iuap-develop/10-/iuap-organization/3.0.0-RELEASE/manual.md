#组织组件概述

## 业务需求 ##

企业系统开发中，经常需要维护企业的组织结构，业务会根据企业的组织进行管理，因此我们提供基本的组织组件，业务可以根据自己的需要进行扩张使用。

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：

```
	<dependency>
      <groupId>com.yonyou.iuap</groupId>
      <artifactId>iuap-organization</artifactId>
      <version>${iuap.modules.version}</version>
    </dependency>
```

${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能说明 ##

组织组件主要是提供组织的核心模型和基本服务，支持通过组织职能的方式对组织进行扩展。

1.	提供组织的核心模型和基本服务
2.	提供组织基本信息的管理；
3.	提供组织职能的管理；
4.	支持通过组织职能的方式对组织进行扩展；
5.	支持对组织基本信息和组织职能的扩展；

# 使用说明 #

## 使用方法

** 1. 	配置spring文件，参考示例工程：organization-applicationContext.xml **

** 2. 执行数据库脚本 **

	依次执行examples项目下sql目录中的dll.sql、index.sql、dml.sql建立数据库并初始化数据

** 3. 组织表结构说明：**

##### （1）组织表org_orgs

<table>
   <tr>
      <td>名称</td>
      <td>代码</td>
      <td>注释</td>
   </tr>
   <tr>
      <td>组织Id</td>
      <td>id</td>
      <td></td>
   </tr>
   <tr>
      <td>组织编码</td>
      <td>code</td>
      <td></td>
   </tr>
   <tr>
      <td>组织名称</td>
      <td>name</td>
      <td></td>
   </tr>
   <tr>
      <td>上级组织Id</td>
      <td>fatherid</td>
      <td></td>
   </tr>
   <tr>
      <td>组织描述</td>
      <td>description</td>
      <td>对组织的简要描述</td>
   </tr>
   <tr>
      <td>租户Id</td>
      <td>tenantid</td>
      <td></td>
   </tr>
   <tr>
      <td>应用系统Id</td>
      <td>sysid</td>
      <td></td>
   </tr>
   <tr>
      <td>内部编码</td>
      <td>innercode</td>
      <td>构建编码树，用于查询某个组织的所有下级</td>
   </tr>
   <tr>
      <td>组织类型一</td>
      <td>orgtype1</td>
      <td>组织类型，目前是十个，支持扩展</td>
   </tr>
   <tr>
      <td>组织类型二</td>
      <td>orgtype2</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型三</td>
      <td>orgtype3</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型四</td>
      <td>orgtype4</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型五</td>
      <td>orgtype5</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型六</td>
      <td>orgtype6</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型七</td>
      <td>orgtype7</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型八</td>
      <td>orgtype8</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型九</td>
      <td>orgtype9</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型十</td>
      <td>orgtype10</td>
      <td></td>
   </tr>
</table>

##### （2）组织类型表org_type

<table>
   <tr>
      <td>名称</td>
      <td>代码</td>
      <td>注释</td>
   </tr>
   <tr>
      <td>组织类型id</td>
      <td>id</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型编码</td>
      <td>code</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型名称</td>
      <td>name</td>
      <td></td>
   </tr>
   <tr>
      <td>租户id</td>
      <td>tenantid</td>
      <td></td>
   </tr>
   <tr>
      <td>应用系统id</td>
      <td>sysid</td>
      <td></td>
   </tr>
   <tr>
      <td>组织类型的字段名</td>
      <td>fieldname</td>
      <td>字段名代码的是组织的类型，如orgtype1</td>
   </tr>
   <tr>
      <td>组织类型的服务类</td>
      <td>serviceclass</td>
      <td>每个组织类型扩展一张表，这个字段用于注册服务类</td>
   </tr>
   <tr>
      <td>组织类型的实体类</td>
      <td>entityclassname</td>
      <td>每个组织类型的实体类，若是组织类型为公司，就是公司的实体类</td>
   </tr>
</table>


## API接口

### IOrgService，主要提供组织相关的接口：

#### 保存组织基本信息

**描述**  

保存组织基本信息

**请求方法**  

com.yonyou.iuap.org.service.IOrgService.save(OrgEntity orgEntity)

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>orgEntity</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织实体，详见OrgEntity</td>
	</tr>
</table>


**返回参数说明**  

无


#### 删除组织基本信息

**描述**  

根据主键删除组织基本信息

**请求方法**  

com.yonyou.iuap.org.service.IOrgService.deleteById(String id);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>id</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织基本信息主键</td>
	</tr>
</table>

**返回参数说明**  

无

#### 根据主键查询组织基本信息

**描述**  

根据主键查询组织基本信息

**请求方法**  

com.yonyou.iuap.org.service.IOrgService.queryById(String id);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>id</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织基本信息主键</td>
	</tr>
</table>

**返回参数说明**  

组织实体 ：OrgEntity



#### 根据组织类型字段查询组织基本信息

**描述**  

根据组织类型字段查询组织基本信息

**请求方法**  

com.yonyou.iuap.org.service.IOrgService.queryOrgsByOrgType(String orgtype)；

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>orgtype</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织类型</td>
	</tr>
</table>

**返回参数说明**  

组织实体列表：List<OrgEntity>


#### 根据多个组织类型字段查询组织基本信息

**描述**  

根据多个组织类型字段查询组织基本信息

**请求方法**  

com.yonyou.iuap.org.service.IOrgService.queryOrgsByOrgTypeArray(String[] orgtypeArray);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>orgtype</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织类型数组，例如：{"orgtype1","orgtype2",...}</td>
	</tr>
</table>

**返回参数说明**  


组织实体列表：List<OrgEntity>



#### 查询上级组织基本信息

**描述**  

根据组织基本信息主键查询上级组织基本信息

**请求方法**  

com.yonyou.iuap.org.service.IOrgService.queryParentOrg(String id);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>id</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织基本信息主键</td>
	</tr>
</table>

**返回参数说明**  

上级组织实体：OrgEntity



#### 查询下级组织基本信息

**描述**  

根据组织基本信息主键查询下级组织基本信息

**请求方法**  

com.yonyou.iuap.org.service.IOrgService.queryChildrenOrgs(String id);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>id</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织基本信息主键</td>
	</tr>
</table>

**返回参数说明**  

包含本级的全部下级组织基本信息：List<OrgEntity>



#### 查询全部组织基本信息

**描述**  

查询全部组织基本信息

**请求方法**  

com.yonyou.iuap.org.service.IOrgService.queryAllOrgs();

**请求方式**  

服务调用  

**请求参数说明**  

无

**返回参数说明**  

全部组织基本信息：List<OrgEntity>



#### 根据ID批量查询组织基本信息

**描述**  

根据ID批量查询组织基本信息

**请求方法**  

com.yonyou.iuap.org.service.IOrgService.queryOrgsByIds(String[] ids);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>ids</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织基本信息主键数组</td>
	</tr>
</table>

**返回参数说明**  

织基本信息列表：List<OrgEntity>

### IOrgTypeService，主要提供组织职能的相关接口：

#### 保存组织职能

**描述**  

保存组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.save(OrgTypeEntity orgTypeEntity);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>OrgTypeEntity</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织职能实体，详见OrgTypeEntity</td>
	</tr>
</table>


**返回参数说明**  

无


#### 删除组织职能

**描述**  

根据ID删除组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.deleteById(String id);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>id</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织职能主键</td>
	</tr>
</table>

**返回参数说明**  

无

#### 根据ID查询组织职能

**描述**  

根据ID查询组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.queryOrgTypebyId(String id);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>id</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织职能主键</td>
	</tr>
</table>

**返回参数说明**  

组织职能实体 OrgTypeEntity

#### 根据编码查询组织职能

**描述**  

根据编码查询组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.queryOrgTypeByCode(String code);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>code</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织职能编码</td>
	</tr>
</table>

**返回参数说明**  

组织职能实体 OrgTypeEntity

#### 根据名称查询组织职能

**描述**  

根据名称查询组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.queryOrgTypeByName(String name);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>name</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织职能名称</td>
	</tr>
</table>

**返回参数说明**  

组织职能实体 OrgTypeEntity

#### 根据组织基本信息职能字段查询组织职能

**描述**  

根据组织基本信息职能字段查询组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.queryOrgTypeByFieldName(String fieldname);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>fieldname</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织职能在基本信息中的字段名称</td>
	</tr>
</table>

**返回参数说明**  

组织职能实体 OrgTypeEntity

#### 根据组织职能的实体类查询组织职能

**描述**  

根据组织职能的实体类查询组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.queryOrgTypeByEntityClassName(String entityclassname);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>entityclassname</td>
		<td>True</td>
		<td>String</td>
		<td>无</td>
		<td>组织职能的对应的实体类的全类名</td>
	</tr>
</table>

**返回参数说明**  

组织职能实体 OrgTypeEntity

#### 根据组织职能服务类查询拥有的组织职能

**描述**  

根据组织基本信息查询组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.queryOrgTypeByServiceClassName(String serviceclass);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>serviceclass</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织职能中注册的服务类</td>
	</tr>
</table>

**返回参数说明**  

组织职能实体 OrgTypeEntity

#### 根据组织职能主键批量查询组织职能

**描述**  

根据组织职能主键批量查询组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.queryOrgTypesByIds(String[] ids);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>ids</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织职能主键数组</td>
	</tr>
</table>

**返回参数说明**  

组织职能实体列表 List<OrgTypeEntity>

#### 根据组织基本信息查询拥有的组织职能

**描述**  

根据组织基本信息查询组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.queryOrgTypeByOrg(OrgEntity orgEntity);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>orgEntity</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织基本信息实体</td>
	</tr>
</table>

**返回参数说明**  

组织职能实体列表 List<OrgTypeEntity>

#### 根据组织基本信息主键查询拥有的组织职能

**描述**  

根据组织基本信息主键查询组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.queryOrgTypeByOrgID(String orgId);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>id</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织基本信息主键</td>
	</tr>
</table>

**返回参数说明**  

组织职能实体列表 List<OrgTypeEntity>

#### 根据组织基本信息判断是否拥有指定的组织职能

**描述**  

根据组织基本信息判断是否拥有指定的组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.isTypeOf(OrgEntity orgEntity, String orgtypeId);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>orgEntity</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织基本信息实体</td>
	</tr>
	<tr>
		<td>orgtypeId</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织职能主键</td>
	</tr>
</table>

**返回参数说明**  

boolean

#### 根据组织基本信息主键判断是否拥有指定的组织职能

**描述**  

根据组织基本信息主键判断是否拥有指定的组织职能

**请求方法**  

com.yonyou.iuap.org.service.IOrgTypeService.isTypeOfByOrgId(String orgId, String orgtypeId);

**请求方式**  

服务调用  

**请求参数说明**  

<table>
	<tr>
		<th>参数字段</th>
		<th>必选</th>
		<th>类型</th>
		<th>长度限制</th>
		<th>说明</th>
	</tr>
	<tr>
		<td>orgId</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织基本信息主键</td>
	</tr>
	<tr>
		<td>orgtypeId</td>
		<td>True</td>
		<td>String[]</td>
		<td>无</td>
		<td>组织职能主键</td>
	</tr>
</table>

**返回参数说明**  

boolean


## 扩展开发说明

#### 1，扩展组织基本信息

例如，我们想要增加描述组织信息的字段，可以扩展组织表org_orgs，修改服务IOrgService的默认实现，在spring中配置即可。

也可在具体的组织类型上增加相应组织类型特有的字段，对应的服务类和实体类自己实现，参照组织职能扩展，对应的pk_org存为该数据同步到组织表中的主键。

#### 2，组织职能扩展

例如，我们想要增加公司这一主职职能，应在org_type中注册一条组织职能的相关信息,注册组织类型对应的服务类与实体类，并进行相应的实现。
<table>
    <tr>
        <td>组织类型id（id）</td>
        <td>组织类型编码（code)</td>
        <td>组织类型名称(name)</td>
		<td>租户id(tenantid)</td>
		<td>应用系统id(sysid)</td>
		<td>组织类型的字段名(fieldname)</td>
		<td>组织类型的服务类(serviceclass)</td>
		<td>组织类型的实体类(entityclassname)</td>
    </tr>
    <tr>
        <td>00229d33-2807-4ee9-9aca-ef9e4311ef46</td>
        <td>corp</td>
        <td>公司(name)</td>
		<td></td>
		<td></td>
		<td>orgtype1</td>
		<td>com.yonyou.iuap.org.service.ICorpService</td>
		<td>com.yonyou.iuap.org.entity.CorpEntity</td>
    </tr>
</table>

最后，要将扩展的组织数据同步到org_orgs中，pk相同，组织表中的orgtype1字段存为'Y'
