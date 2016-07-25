#公式组件概述#

公式组件是以接口的方式提供算数、字符串运算能力的组件。使用者定义公式和变量，公式组件根据公式和变量值进行运算并返回运算结果。

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：

```
	<dependency>
	  <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-formula</artifactId>
	  <version>${iuap.modules.version}</version>
	</dependency>
```
${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能说明 ##

1.	支持公式定义和执行；
2.	支持公式批量执行；
3.	提供数学运算、字符串运算能力
4.	支持自定义函数功能扩展公式；

# 使用说明 #

## 说明简介

下文主要是对公式组件的基本使用进行简要说明，对公式函数中的一些特殊情况加以说明，并对公式函数自定义扩展功能予以介绍

##调用说明##

通过FormulaParseFather执行公式。

示例：
```
	    String formula = "a->abs(data)";
		FormulaParseFather f = new FormulaParse();;
		f.setExpress(formula);
		f.addVariable("data", new Object[] { 1, -2, 0 });
		Object[][] result = f.getValueOArray();
		assertEquals("应该相等", 1, result[0][0]);
		assertEquals("应该相等", 2, result[0][1]);
		assertEquals("应该相等", 0, result[0][2]);
```	

##函数介绍 ##

### 1.字符串函数 ###

#### 1.1 charat(string,index) ####

公式含义

得到字符串string中第index个字符

对应类：nc.vo.pub.formulaset.function.CharAt

类型返回值表

<table>
   <tr>
      <td>参数string</td>
      <td>参数index</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>Number</td>
      <td>java.lang.String, index转换成int数值，与Java中charAt()结果相同，当index大于string长度时，抛出异常</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.String</td>
      <td>java.lang.String，index必须可以直接从String转成int，如”1”(但“1.22”不行，转int失败抛异常)，转换后与上一条执行结果相同。</td>
   </tr>
   
</table>


#### 1.2 endswith(string, end) ####

公式含义

判断字符串string是否以字符串end结尾

对应类：nc.vo.pub.formulaset.function.EndsWith

类型返回值表

<table>
   <tr>
      <td>参数string</td>
      <td>参数end</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.String</td>
      <td>java.lang.Double,与Java中String类方法endsWith返回结果相同，只不过返回Boolean</td>
   </tr>
   <tr>
      <td>null</td>
      <td>除null以外任何对象</td>
      <td>Boolean.FALSE</td>
   </tr>
   <tr>
      <td>除null以外任何对象</td>
      <td>null</td>
      <td>Boolean.FALSE</td>
   </tr>
   <tr>
      <td>null</td>
      <td>null</td>
      <td>Boolean.TRUE</td>
   </tr>
  
</table>



#### 1.3 equalsIgnoreCase(string1, string2) ####
公式含义

判断忽略大小写字符串string1是否与字符串string2相等
	
对应类：nc.vo.pub.formulaset.function.EqualsIgnoreCase

类型返回值表
<table>
   <tr>
      <td>参数string</td>
      <td>参数end</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.String</td>
      <td>Integer, 与Java中String类方法equalsIgnoreCase返回结果相同，只不过返回Integer(true对应1，false对应-1)</td>
   </tr>
   <tr>
      <td>null</td>
      <td>除null以外任何对象</td>
      <td>new Integer(-1)</td>
   </tr>
   <tr>
      <td>除null以外任何对象</td>
      <td>null</td>
      <td>new Integer(-1)</td>
   </tr>
   <tr>
      <td>null</td>
      <td>null</td>
      <td>new Integer(1)</td>
   </tr>
</table>

#### 1.4	indexOf(st1, st2)  ####

公式含义

判断字符串st1中第一个字符串st2所在的位置,比如lastIndexOf("HI,UAP2006, UAP","UAP")返回3.

对应类：nc.vo.pub.formulaset.function.IndexOf

类型返回值表
<table>
   <tr>
      <td>参数st1</td>
      <td>参数st2</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.String</td>
      <td>Integer, 与Java中String类方法indexOf返回结果相同,如果不存在返回-1</td>
   </tr>
  
</table>
注：参数都不能为null

####1.5 isEmpty(str) ####

公式含义
用于判断变量是否为空,包括空串("")及空值(null)
	对应类：nc.vo.pub.formulaset.function.IsEmpty

类型返回值表
<table>
   <tr>
      <td>参数str</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>Boolean；如果str为””,返回Boolean.TRUE,否则返回Boolean.FALSE</td>
   </tr>
   <tr>
      <td>null</td>
      <td>Boolean.TRUE</td>
   </tr>
   <tr>
      <td>其他任何对象</td>
      <td>Boolean.FALSE</td>
   </tr>
  
</table>

#### 1.6 lastIndexOf(st1, st2)  ####
公式含义

判断字符串st1中最后一个字符串st2所在的位置,比如lastIndexOf("HI,UAP2006,UAP","UAP")返回11.

对应类：nc.vo.pub.formulaset.function.LastIndexOf

类型返回值表

<table>
   <tr>
      <td>参数st1</td>
      <td>参数st2</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.String</td>
      <td>Integer, 与Java中String类方法lastIndexOf返回结果相同,如果不存在返回-1</td>
   </tr>
  
</table>

#### 1.7 left(st, index)  ####

公式含义

求字符串st左边前index个字符组成的字符串
对应类：nc.vo.pub.formulaset.function.Left

类型返回值表

<table>
   <tr>
      <td>参数st</td>
      <td>参数index</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.Object(其toString()方法一定要可以parse为int)</td>
      <td>java.lang.String, 与Java中String类方法subString(0,index)返回结果相同,如果index>st字符串的长度，抛异常</td>
   </tr>
  
</table>

#### 1.8 leftStr(st,len,defaultStr)  ####

公式含义

求字符串st左边前len个字符组成的字符串，如果字符串长度小于len，则用defaultStr补齐,比如leftStr("abc",6,"@")将返回abc@@@.

对应类：nc.vo.pub.formulaset.function.LeftStr

类型返回值表
<table>
   <tr>
      <td>参数st</td>
      <td>参数len</td>
      <td>参数defaultStr</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String，长度不能为0</td>
      <td>java.lang.Object，但其toString()方法要能转换为int值</td>
      <td>java.lang.String，长度不能为0</td>
      <td>java.lang.String，将str的字符串值从左侧进行截取，如果长度不够len，则用defaultStr补齐</td>
   </tr>
  
</table>

#### 1.9 length(st)  ####
公式含义

求字符串st的长度

对应类：nc.vo.pub.formulaset.function.Length

类型返回值表

<table>
   <tr>
      <td>参数st</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>Integer, 与String的length方法返回结果相同</td>
   </tr>
   <tr>
      <td>null</td>
      <td>0</td>
   </tr>
  
</table>

#### 1.10 mid(String st, int start, int end)  ####

公式含义

求字符串st左边前第start个字符至第end个字符之间的字符串

对应类：nc.vo.pub.formulaset.function.Mid

类型返回值表

<table>
   <tr>
      <td>参数st</td>
      <td>参数start</td>
      <td>参数end</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String，长度不能为0</td>
      <td>java.lang.Object，但其toString()方法要能转换为int值</td>
      <td>java.lang.Object，但其toString()方法要能转换为int值</td>
      <td>java.lang.String，与String中的方法subString(start, end)</td>
   </tr>
 
</table>

#### 1.11 right(String st, int index)  ####
公式含义

求字符串st右边前index个字符组成的字符串
对应类：nc.vo.pub.formulaset.function.Right

类型返回值表
<table>
   <tr>
      <td>参数st</td>
      <td>参数index</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.Object(其toString()方法一定要可以parse为int)</td>
      <td>java.lang.String, 与Java中String类方法subString(st.length()-index)返回结果相同,如果index>st字符串的长度，抛异常</td>
   </tr>
 
</table>


#### 1.12 rightStr(st,len,defaultStr)  ####

公式含义

求字符串st右边后len个字符组成的字符串，如果字符串长度小于len，则用defaultStr补齐,比如rightStr("abc",6,"@")将返回abc@@@.
对应类：nc.vo.pub.formulaset.function.RightStr

类型返回值表
<table>
   <tr>
      <td>参数st</td>
      <td>参数len</td>
      <td>参数defaultStr</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String，长度不能为0</td>
      <td>java.lang.Object，但其toString()方法要能转换为int值</td>
      <td>java.lang.String，长度不能为0</td>
      <td>java.lang.String，将str的字符串值从右侧进行截取，如果长度不够len，则用defaultStr补齐</td>
   </tr>
  
</table>

#### 1.13 startsWith(String st, String start)  ####

公式含义

判断字符串st是否以字符串start开头

对应类：nc.vo.pub.formulaset.function.StartsWith
类型返回值表
<table>
   <tr>
      <td>参数string</td>
      <td>参数end</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.String</td>
      <td>java.lang.Double,与Java中String类方法startsWith返回结果相同，只不过返回Boolean</td>
   </tr>
   <tr>
      <td>null</td>
      <td>除null以外任何对象</td>
      <td>Boolean.FALSE</td>
   </tr>
   <tr>
      <td>除null以外任何对象</td>
      <td>null</td>
      <td>Boolean.FALSE</td>
   </tr>
   <tr>
      <td>null</td>
      <td>null</td>
      <td>Boolean.TRUE</td>
   </tr>
  
</table>

#### 1.14 toLowerCase(String st)  ####

公式含义

求字符串st的小写形式,比如toLowerCase("Abc")返回"abc"。

对应类：nc.vo.pub.formulaset.function.ToLowerCase
类型返回值表
<table>
   <tr>
      <td>参数st</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.String, 与String的toLowerCase方法返回结果相同</td>
   </tr>
  
</table>
注：参数不能为null。

#### 1.15 toString(obj)  ####

公式含义

将对象obj转换为本解析器可识别的字符串形式。

对应类：nc.vo.pub.formulaset.function.ToString

类型返回值表
<table>
   <tr>
      <td>参数st</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>null</td>
      <td>java.lang.String, “null”</td>
   </tr>
   <tr>
      <td>nc.vo.pub.formulaset.util.NullZeroNumber</td>
      <td>java.lang.String,””</td>
   </tr>
   <tr>
      <td>其他的java.lang.Object</td>
      <td>java.lang.String,返回toString()方法的内容</td>
   </tr>
  
</table>

#### 1.16 toUpperCase(String st)  ####

公式含义

求字符串st的大写形式。

对应类：nc.vo.pub.formulaset.function.ToUpperCase

类型返回值表

<table>
   <tr>
      <td>参数st</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.String, 与String的toUpperCase方法返回结果相同</td>
   </tr>
  
</table>
注：参数不能为null。

#### 1.17 trimZero(value, [decimal]) ####

公式含义

剪除字符串或数字str的末尾0值, var:表示待截取的值，可以为数值也可以为字符，若字符型不能转换为数值则返回原值(如"8.023a")。 
[decimal]:可选项,表示精度值(即小数点后的位数). 
trimZero("8.023000"); //return "8.023" 
trimZero(8.023000); //return 8.023 
trimZero("8.023000a"); //return "8.023000a" 
trimZero("8.023000",2); //return "8.02"

对应类：nc.vo.pub.formulaset.function.TrimZero。

类型返回值表
<table>
   <tr>
      <td>参数value</td>
      <td>参数decimal</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>java.lang.String</td>
      <td>java.lang.Object，但其toString方法要能转换成int值</td>
      <td>如果String可以直接转成Double，返回具体精度的Double，否则直接返回value。特别地，当value长度为0时返回Double(0.0)</td>
   </tr>
   <tr>
      <td>Double</td>
      <td>同上</td>
      <td>返回具体精度的Double</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>同上</td>
      <td>将value转换为double后，再按照精度转换成Double</td>
   </tr>
   <tr>
      <td>任何其他java.lang.Object</td>
      <td>同上</td>
      <td>直接返回value</td>
   </tr>
   <tr>
      <td>null</td>
      <td>同上</td>
      <td>null</td>
   </tr>
 
</table>
注：参数decimal可以不写，此时默认为-1（int），如果设置decimal的值，则其不能为null。

### 2.数学函数 ###

#### 2.1 abs(num)  ####

公式含义
求数num的绝对值


对应类：nc.vo.pub.formulaset.function.Abs

类型返回值表
<table>
   <tr>
      <td>参数num</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Integer</td>
      <td>Integer, 与Math.abs(int)方法返回结果相同</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double, 与Math.abs(double)方法返回结果相同</td>
   </tr>
   <tr>
      <td>nc.vo.formulaset.coretype.Complex</td>
      <td>Double,与Complex返回结果相同</td>
   </tr>
   
</table>
注：参数值不能为null。

#### 2.2 acos(x) ####

公式含义

返回一个弧度x的反余弦(arccos),弧度值在0到Pi之间

对应类：nc.vo.pub.formulaset.function.ArcCosine

类型返回值表
<table>
   <tr>
      <td>参数num</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double, 与Math.acos(double)方法返回结果相同</td>
   </tr>
   <tr>
      <td>nc.vo.formulaset.coretype.Complex</td>
      <td>Double,与Complex.acos():返回结果相同</td>
   </tr>
 
</table>
注：参数不能为null。Complex.acos():acos(z)  =  -i * log( z + i * sqrt(1 - z*z) )。

#### 2.3 add(num1,num2) ####

公式含义

用于高精度加法运算。

对应类：nc.vo.pub.formulaset.function.AddBigNumber。

类型返回值表
<table>
   <tr>
      <td>参数num1</td>
      <td>参数num2</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Double</td>
      <td>null</td>
      <td>Double: num1</td>
   </tr>
   <tr>
      <td></td>
      <td>Double</td>
      <td>Double:num1+num2</td>
   </tr>
   <tr>
      <td></td>
      <td>Number</td>
      <td>Double:num1+num2</td>
   </tr>
   <tr>
      <td>null</td>
      <td>Double</td>
      <td>Double:num2</td>
   </tr>
   <tr>
      <td>Number</td>
      <td></td>
      <td>Double:num1+num2</td>
   </tr>
   
</table>
注：其他的情况下，将会用普通+的方式进行运算

#### 2.4 asin(x) ####
公式含义

返回一个弧度x的反正弦(arcsin),弧度值在-Pi/2到Pi/2之间

对应类：nc.vo.pub.formulaset.function.ArcSine

类型返回值表
<table>
   <tr>
      <td>参数num</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double, 与Math.asin(double)方法返回结果相同</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Double,与Complex.asin():返回结果相同</td>
   </tr>
  
</table>
注：参数不能为null。
Complex.asin(): asin(z)  =  -i * log(i*z + sqrt(1 - z*z))。

#### 2.5 atan(x) ####

公式含义

返回一个弧度x的反正切值,弧度值在-Pi/2到Pi/2之间

对应类：nc.vo.pub.formulaset.function.ArcTangent

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double, 与Math.atan(double)方法返回结果相同</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Double,与Complex.atan():返回结果相同</td>
   </tr>
   
</table>
注：参数不能为null。
Complex.atan():atan(z) = -i/2 * log((i-z)/(i+z))。

#### 2.6 cos(x) ####

公式含义

返回给定角度x的余弦值
对应类：nc.vo.pub.formulaset.function.Cosine

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double, 与Math.cos(double)方法返回结果相同</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Double,与Complex.cos():返回结果相同</td>
   </tr>
  
</table>
注：参数不能为null。
Complex.cos():cos(z)  =  ( exp(i*z) + exp(-i*z) ) / 2。


#### 2.7 div(num1,num2) ####

公式含义

用于高精度除法运算

对应类：nc.vo.pub.formulaset.function.DivideBigNumber

类型返回值表

<table>
   <tr>
      <td>参数num1</td>
      <td>参数num2</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Double</td>
      <td>null</td>
      <td>Double: num1</td>
   </tr>
   <tr>
      <td></td>
      <td>Double</td>
      <td>Double:num1.div(num2)</td>
   </tr>
   <tr>
      <td></td>
      <td>Number</td>
      <td>Double: num1.div(num2)</td>
   </tr>
   <tr>
      <td>null</td>
      <td>Double</td>
      <td>Double:num2</td>
   </tr>
   <tr>
      <td>Number</td>
      <td></td>
      <td>Double: num1.div(num2)</td>
   </tr>
  
</table>
注：其他情况下，会使用/操作进行运算

#### 2.8 exp(x) ####

公式含义

e的x次方

对应类：nc.vo.pub.formulaset.function.Exp

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double, 与Math.exp(x.doubleValue())方法返回结果相同</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex, new Complex(mod*Math.cos(y),mod*Math.sin(y))</td>
   </tr>
  
</table>
注：参数不能为null

#### 2.9 int(x) ####

公式含义

将变量转换为int类型；

对应类：nc.vo.pub.formulaset.function.IntValue

类型返回值表

<table>
   <tr>
      <td>参数x</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Integer, new Integer(Math.round(x.floatValue()))</td>
   </tr>
   <tr>
      <td>String</td>
      <td>Integer, new Integer (Math.round(new Double(x.toString()).floatValue))</td>
   </tr>
  
</table>

注：参数不能为null

#### 2.10 ln(x) ####

公式含义

返回给定数值x的自然对数；

对应类：nc.vo.pub.formulaset.function.NaturalLogarithm

类型返回值表

<table>
   <tr>
      <td>参数x</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Complex, new Complex(param.doubleValue()).log()</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex, x.log()</td>
   </tr>
   
</table>
注：参数不能为null

#### 2.11 log(x) ####

公式含义

返回给定数n的以十为底的对数；

类型返回值表

<table>
   <tr>
      <td>参数x</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Complex, new Complex(x.doubleValue()).log().div(new Complex(Math.log(10), 0))</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex, x.log().div(new Complex(Math.log(10),0))</td>
   </tr>
   <tr>
      <td></td>
   </tr>
</table>
注：参数不能为null
 
#### 2.12 max(x, y) ####

公式含义

求数字x,y两者中的最大值

对应类：nc.vo.pub.formulaset.function.Max

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>参数y</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Number</td>
      <td>Number,x.doubleValue()与y.doubleValue()比较，>返回x，其余返回y</td>
   </tr>
   <tr>
      <td>Comparable</td>
      <td>Object</td>
      <td>x.compareTo(y) ,>0返回x,<0返回y</td>
   </tr>
   <tr>
      <td>Object</td>
      <td>Comparable</td>
      <td>-y.compareTo(x) ,>0返回x,<0返回y</td>
   </tr>
  
</table>

注：参数不能为null，Comparable：java.lang.Comparable。

#### 2.13 min(x, y) ####

公式含义

求x,y两者中的最小值；

对应类：nc.vo.pub.formulaset.function.Min

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>参数y</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Number</td>
      <td>Number, x.doubleValue()与y.doubleValue()比较，>返回x，其余返回y</td>
   </tr>
   <tr>
      <td>Comparable</td>
      <td>Object</td>
      <td>x.compareTo(y)进行比较，>0返回y,<0返回x</td>
   </tr>
   <tr>
      <td>Object</td>
      <td>Comparable</td>
      <td>-y.compareTo(x)进行比较，>0返回y,<0返回x</td>
   </tr>
  
</table>

#### 2.14 mul(num1,num2) ####

公式含义

用于高精度乘法运算；

对应类：nc.vo.pub.formulaset.function.MultiplyBigNumber

类型返回值表
<table>
   <tr>
      <td>参数num1</td>
      <td>参数num2</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Double</td>
      <td>null</td>
      <td>Double: num1</td>
   </tr>
   <tr>
      <td></td>
      <td>Double</td>
      <td>Double:num1.multiply(num2)</td>
   </tr>
   <tr>
      <td></td>
      <td>Number</td>
      <td>Double: num1. multiply (num2)</td>
   </tr>
   <tr>
      <td>null</td>
      <td>Double</td>
      <td>Double:num2</td>
   </tr>
   <tr>
      <td>Number</td>
      <td></td>
      <td>Double: num1. multiply (num2)</td>
   </tr>
  
</table>

注：其他情况下，会使用*操作进行运算

#### 2.15 round(double num, int index) ####

公式含义

对num保留index位小数(四舍五入)；

对应类：nc.vo.pub.formulaset.function.Round

类型返回值表

<table>
   <tr>
      <td>参数x</td>
      <td>参数y</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>String</td>
      <td>Number</td>
      <td>new Double(x).setScale(y,Double.ROUND_HALF_UP)</td>
   </tr>
   <tr>
      <td>Double</td>
      <td>Number</td>
      <td>new Double(x).setScale(y,Double.ROUND_HALF_UP)</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Number</td>
      <td>new Double(x). setScale(y,Double.ROUND_HALF_UP)</td>
   </tr>
  
</table>
注：返回类型均为Double，参数不能为null

#### 2.16 sgn(num) ####

公式含义

当数num大于0时,返回1,等于0时,返回0,小于0时返回-1；

对应类：nc.vo.pub.formulaset.function.Sgn

类型返回值表
<table>
   <tr>
      <td>参数num</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double,判断num.doubleValue()与0之间的关系，>时返回1，=时返回0， <时返回-1</td>
   </tr>
  
</table>
注：参数不能为null

#### 2.17 sin(x) ####

公式含义

返回给定角度x的正弦值；

对应类：nc.vo.pub.formulaset.function.Sine

类型返回值表
<table>
   <tr>
      <td>参数num</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>new Double(Math.sin(((Number)param).doubleValue()))</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex，((Complex)param).sin()</td>
   </tr>
  
</table>
注：参数不能为null

#### 2.18 sqrt(x) ####

公式含义

返回数值x的平方根；

对应类：nc.vo.pub.formulaset.function.SquareRoot

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>x.doubleValue()<0，返回new Complex(value).sqrt()，即Complex；</td>
   </tr>
   <tr>
      <td>x.doubleValue()<0，返回new Double(Math.sqrt(x))，即Double</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex，((Complex)param).sqrt()</td>
   </tr>
   
</table>
注：参数不能为null

#### 2.19 sub(num1,num2) ####

公式含义

用于高精度减法运算；

类型返回值表
<table>
   <tr>
      <td>参数num1</td>
      <td>参数num2</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Double</td>
      <td>null</td>
      <td>Double: num1</td>
   </tr>
   <tr>
      <td></td>
      <td>Double</td>
      <td>Double:num1-num2</td>
   </tr>
   <tr>
      <td></td>
      <td>Number</td>
      <td>Double:num1-num2</td>
   </tr>
   <tr>
      <td>null</td>
      <td>Double</td>
      <td>Double:num2</td>
   </tr>
   <tr>
      <td>Number</td>
      <td></td>
      <td>Double:num1-num2</td>
   </tr>
  
</table>

注：其他的情况下，将会用普通-的方式进行运算


#### 2.20 tan(x) ####
公式含义

返回给定角度x的正切值；

对应类：nc.vo.pub.formulaset.function.Tangent

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double,new Double(Math.tan(((Number)param).doubleValue()))</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex，((Complex)param).tan()</td>
   </tr>
 
</table>
注：参数类型不能为null

#### 2.21 toNumber(String st) ####

公式含义

将字符串st转换为本解析器可识别的数字,比如toNumber("45.0")将返回一个数字型45.0,经过转化后可参与各种数值计算.

类型返回值表
<table>
   <tr>
      <td>参数st</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>String</td>
      <td>st.toString()==null或trim().length()==0，返回new Double(0);</td>
   </tr>
   <tr>
      <td>new Double(st.toString().trim())</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Number，st</td>
   </tr>
  
</table>
注：参数类型不能为null

#### 2.22 zeroifnull(var) ####

公式含义

表示如果var为空将返回0；

对应类：nc.vo.pub.formulaset.function.ZeroIfNull

类型返回值表
<table>
   <tr>
      <td>参数var</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>null</td>
      <td>new Double(0)</td>
   </tr>
   <tr>
      <td>String</td>
      <td>如果var.length==0，返回new Double(0)</td>
   </tr>
   <tr>
      <td>其他情况</td>
      <td>var</td>
   </tr>
  
</table>
注：参数类型不能为null

#### 2.23 acosh(param) ####

公式含义

acosh(z)  =  log(z + sqrt(z*z - 1))

对应类：nc.vo.pub.formulaset.function.ArcCosineH

类型返回值表
<table>
   <tr>
      <td>参数param</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Complex: new Complex(((Number)param).doubleValue(),0.0).acosh()</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex: ((Complex)param).acosh()</td>
   </tr>
 
</table>


#### 2.24 angle(x,y) ####

公式含义

  以弧度为单位计算并返回点y/x的角度。

对应类：nc.vo.pub.formulaset.function.Angle

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>参数y</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Number</td>
      <td>Double: new Double(Math.atan2(x.doubleValue(), y.doubleValue()))</td>
   </tr>
  
</table>

#### 2.25 asinh(param) ####

公式含义

asinh(z)  =  log(z + sqrt(z*z + 1))

对应类：nc.vo.pub.formulaset.function.ArcSineH

类型返回值表
<table>
   <tr>
      <td>参数param</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Complex: new Complex(((Number)param).doubleValue(),0.0).asinh()</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex: ((Complex)param).asinh()</td>
   </tr>
  
</table>


#### 2.26 atanh(param) ####

公式含义

asinh(z)  =  log(z + sqrt(z*z + 1))

对应类：nc.vo.pub.formulaset.function.ArcTanH

类型返回值表
<table>
   <tr>
      <td>参数param</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Complex: new Complex(((Number)param).doubleValue(),0.0).atanh()</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex: ((Complex)param).atanh()</td>
   </tr>
  
</table>


#### 2.27 cosh(param) ####

公式含义

cosh(z)  =  ( exp(z) + exp(-z) ) / 2

对应类：nc.vo.pub.formulaset.function.CosineH

类型返回值表
<table>
   <tr>
      <td>参数param</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double: new Double((Math.exp(param.doubleValue()) + Math.exp(-value))/2)</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex: ((Complex)param).cosh()</td>
   </tr>
   
</table>

#### 2.28 mod(x, y) ####

公式含义

求模运算

对应类：nc.vo.pub.formulaset.function.Modulus

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>参数y</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Integer</td>
      <td>Integer</td>
      <td>Integer: x.intValue()%y.intValue()</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Number</td>
      <td>Double: x.doubleValue()%y.doubleValue()</td>
   </tr>
  
</table>


#### 2.29 rand() ####

公式含义

生成随机数

对应类：nc.vo.pub.formulaset.function.Random


类型返回值
返回new Double(Math.random())


#### 2.30 sinh(param) ####

公式含义

sinh(z)  =  ( exp(z) - exp(-z) ) / 2

对应类：nc.vo.pub.formulaset.function.SineH

类型返回值表
<table>
   <tr>
      <td>参数param</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double: new Double((Math.exp(param.doubleValue()) - Math.exp(-value))/2)</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex: ((Complex)param).sinh()</td>
   </tr>
 
</table>


#### 2.31 sum(x,y) ####

公式含义

计算两个数的和

对应类：nc.vo.pub.formulaset.function.Sum

类型返回值表
<table>
   <tr>
      <td>参数x</td>
      <td>参数y</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Integer</td>
      <td>Integer</td>
      <td>Integer</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Number</td>
      <td>Number</td>
   </tr>
  
</table>


#### 2.32 tanh(param) ####

公式含义

tanh(z)  =  sinh(z) / cosh(z)

对应类：nc.vo.pub.formulaset.function.TanH

类型返回值表
<table>
   <tr>
      <td>参数param</td>
      <td>返回值</td>
   </tr>
   <tr>
      <td>Number</td>
      <td>Double: value=param.doubleValue(); new Double((Math.exp(value)-Math.exp(-value))/ (Math.pow(Math.E,value) +Math.pow(Math.E,-value)))</td>
   </tr>
   <tr>
      <td>Complex</td>
      <td>Complex: ((Complex)param).tanh()</td>
   </tr>
  
</table>


## 自定义公式扩展说明 ##

公式提供自定义函数功能。自定义函数配置文件目录如下：

WEB-INF\classes\formulaconfig

配置文件格式如下：

```
	<?xml version="1.0" encoding="utf-8"?>
	<formula-array>
	<formula>
	  <customType>0</customType>
	  <functionName>FORMATADDRESS</functionName>
	  <functionClass>nc.ui.bd.pubinfo.address.FormatAddress</functionClass>
	</formula>
	</formula-array>
```

自定义公式的实现方法可以参见nc.vo.pub.formulaset.function.Max这个类
