# 公式服务组件概述 #

公式服务组件是以Rest服务的方式提供公式运算能力的组件。公式服务组件提供算数、字符串运算能力。使用者定义公式和变量，公式组件根据公式和变量值进行运算并返回运算结果。

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：
```
    <dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-formula-service</artifactId>
	  <version>${iuap.modules.version}</version>
	  <type>war</type>
	 </dependency>
```
${iuap.modules.version} 为平台在maven私服上发布的组件的version。
使用者把War包部署到Tomcat上，启动服务即可。

## 功能说明 ##

1.	支持公式定义和执行；
2.	支持公式批量执行；
3.	提供数学运算、字符串运算能力
4.	支持自定义函数功能扩展公式；
5.	以Rest服务的方式提供公式运算能力


# 使用说明

## 提供的功能说明：

1 提供基本类型、数学函数、字符串函数、日期函数等基本功能。
2 提供自定义函数扩展功能。
3 本组件提供通过resetAPI的方式直接执行公式的能力

## 部署说明：

### 1:部署公式服务到web服务器

下载 iuap-formula-service.war 作为独立应用部署


## 调用示例

### 1:公式Rest服务的get调用

服务接口：

iuap-formula-service/formula/execute4get?formula={"variables":{"d1":3,"d2":4,"i1":6},"formulas":["f1->d1*d2*i1"]}

参数以json 格式传入

返回值格式如下：{"f1":[72]}

### 2:公式Rest服务的post调用

服务接口：
/iuap-formula-service/formula/execute

参数格式：

String param ="{\"formulas\":[\"f1->d1*d2*i1\"],\"variables\":{\"d1\":3,\"d2\":4,\"i1\":6}}";

返回值格式如下：{"f1":[72]}



## API接口

#### 公式执行get方式

**请求方法**  

/formula/execute4get

**请求方式**  

get

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
    <td>formula</td>
    <td>True</td>
    <td>String</td>
    <td>无</td>
    <td>执行公式文本，例如"{\"formulas\":[\"f1->d1*d2*i1\"],\"variables\":{\"d1\":3,\"d2\":4,\"i1\":6}}"</td>
  </tr>
</table>

**返回参数说明**  

返回：map格式的json数据，key－变量名称 value－执行结果（结果可以为数组），例如{"f1":[72]}

#### 公式执行post方式

**请求方法**  

/formula/execute

**请求方式**  

post

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
    <td>formula</td>
    <td>True</td>
    <td>String</td>
    <td>无</td>
    <td>执行公式文本，例如"{\"formulas\":[\"f1->d1*d2*i1\"],\"variables\":{\"d1\":3,\"d2\":4,\"i1\":6}}"</td>
  </tr>
</table>

**返回参数说明**  

返回：map格式的json数据，key－变量名称 value－执行结果（结果可以为数组），例如{"f1":[72]}




## 其他使用说明

提供哪些公式以及具体公式使用方法，包括自定义函数扩展功能，请参见iuap-formula组件文档说明文档
