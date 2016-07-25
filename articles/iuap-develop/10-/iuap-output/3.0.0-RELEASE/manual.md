
#打印组件概述#

## 业务需求 ##

业务系统通过WEB方式提供应用时，由于各个浏览器厂商正在屏蔽插件，所以不能通过传统的浏览器插件调用客户端实现打印功能。这就要求我们实现能在浏览器上实现打印功能。

##解决方案##

iuap-output组件提供了界面直接打印和HTML模板打印两种方式。

* 界面直接打印（即调用js函数完成指定区域的打印）   

* HTML模板方式打印（即通过定义html的模板通过后台接口得到对应的json对象，将json数据导入到html模板中，形成打印界面）

## 功能说明 ##

1.	提供统一的web打印能力；
2.	支持界面直接打印；
3.	支持HTML模板方式打印


# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：
```
	<dependency>
      <groupId>com.yonyou.iuap</groupId>
	  <artifactId>iuap-output</artifactId>
	  <version>${iuap.modules.version}</version>
    </dependency>
```
${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能说明 ##

组织组件主要通过生成pdf的方式提供对于浏览器的打印功能。 

# 使用说明 #

##组件配置##

**在工程中要打印的jsp文件中引入打印的PrintModel.js**
```
	 <!-- 需要jQuery支持 -->
	 <script type="text/javascript" src="resources/js/jquery-2.1.4.min.js"></script>
	 <!-- 界面直接打印组件JS -->
	 <script type="text/javascript" src="resources/js/PrintModel.js"></script>
    
    Js函数说明：  
	    printByElementId(e_id)；// 打印e_id元素的区域
	    printByElementIds（e_ids）// 打印e_ids元素的区域，e_ids为一个数组
	    printByElementCss（e_css）// 打印css样式为e_css元素区域
	    printByElementCsses（e_csses）；// 打印css样式为e_csses元素区域，e_csses为数组 
```	

## 工程样例 ##

### 界面直接打印 ###

1. 前台页面引入组件*PrintModel.js*  
```
	<!-- 需要jQuery支持 -->
	<script type="text/javascript" src="resources/js/jquery-2.1.4.min.js"></script>
	<!-- 界面直接打印组件JS -->
	<script type="text/javascript" src="resources/js/PrintModel.js"></script>
```

2.调用相应的JS函数，传入打印区域的ID或样式，即可打印指定的区域 

```
>    	  function printDirectCurrentPage() {
>     		Print.printByElementIds([ "printarea" ]);
>     	}   
> 
>        <div class="container" id="printarea">
> 			<div class="orderlist">
> 				<table class="ordertable">
> 					<tr>
> 						<td>订单编号：<span>20150920001</span></td><td>订购时间：<span>2015-09-12 12:45:45</span></td>
> 					</tr>
> 					<tr>
> 						<td>客户姓名：<span>客户姓名</span></td><td>联系方式：<span>13436989458</span></td>
> 					</tr>
> 					<tr>
> 						<td colspan="2">配送地址：<span>北京市海淀区北清路68号用友软件园</span></td>
> 					</tr>
> 				</table>
> 			</div>
> 			<div class="goodslist">
> 				<table class="goodstable">
> 					<tr>
> 						<th width="10%">商品编号</th>
> 						<th width="70%">商品名称</th>
> 						<th width="10%">数量</th>
> 						<th width="10%">商品金额</th>
> 					</tr>
> 					<tr>
> 						<td width="10%">000001</td>
> 						<td width="70%">图解CSS3：核心技术与案例实战 </td>
> 						<td width="10%">1</td>
> 						<td width="10%">￥57.30</td>
> 					</tr>
> 				</table>
> 			</div>
> 			<div class="totolAmount">
> 				订单支付金额：<span>￥57.30</span>
> 			</div>
> 		</div>
```

### HTML模板打印 ###

1. 前台页面引入组件*PrintModel.js*  
2. 
```
		<!-- 需要jQuery支持 -->
			<script type="text/javascript" src="resources/js/jquery-2.1.4.min.js"></script>
			<!-- 界面直接打印组件JS -->
			<script type="text/javascript" src="resources/js/PrintModel.js"></script>
```

2. HTML模板方式时需要在自己的js文件中实现函数getStyle(返回css样式定义字符串)、getTemplate(返回模板定义字符串)、getJsonData(返回模板需要的json对象)方法，参考orderdetail.js中的实现方法。 

```
>     function getStyle1(){
	    var style = ' .container {border: solid 1px #aaaaaa;min-width:860px;font-size: 12px;font-family: "宋体";text-align: left;}';
	    style += ' @media print {.printstyle {width: 650px;display: inline-block;}}';
	    style += ' .logo {height: 90px;margin-bottom:20px;border-bottom: solid 1px #aaaaaa;}';
		style += ' .orderlist {font-size: 12px;}';
		style += ' .orderlist table {width: 60%;}';
		style += ' .orderlist TD {font-weight: bold;padding: 3px;}';
		style += ' .orderlist TD SPAN {font-weight: normal;}';
		style += ' .goodslist {font-size: 12px;padding: 1px;}';
		style += ' .goodslist table {width: 100%;border-collapse: collapse;border: solid 1px #000000;}';
		style += ' .goodslist table th, .goodslist table td {border: solid 1px #000000;padding: 3px;}';
		style += ' .goodslist table th {text-align: center;font-weight: normal;}';
		style += ' .totolAmount {text-align: right;font-size: 14px;font-weight: bold;padding-top: 10px;padding-bottom: 10px;padding-right: 9px;}';
		style += ' .totolAmount span {color: #ff0000;}';
		return style;
	}

>     function getJsonData() {
	    var obj = {
		no : "20150920001",
		buydate : "2015-09-12 12:45:45",
		customername : "刘XX",
		phoneno : "134XXXXXXXX",
		customeraddr : "北京市海淀区北清路68号用友软件园",
		totalamount : '57.30',
		goodslist : [ {
			no : "000001",
			name : "图解CSS3：核心技术与案例实战",
			count : "1",
			amount : '57.30'
			} ]
		};
		return obj;
	}

 >     function getTemplate1(){
		var template = '<div class="container">';
		template += '<div class="orderlist">';
		template += '<table class="ordertable">';
		template += '<tr><td>订单编号：<span><%=printdata.no%></span></td><td>购买日期：<span><%=printdata.buydate%></span></td></tr>';
		template += '<tr><td>客户姓名：<span><%=printdata.customername%></span></td><td>联系方式：<span><%=printdata.phoneno%></span></td></tr>';
		template += '<tr><td colspan="2">配送地址：<span><%=printdata.customeraddr%></span></td></tr>';
		template += '</table></div>';
		template += '<div class="goodslist">';
		template += '<table class="goodstable">';
		template += '<tr><th width="10%">商品编号</th><th width="70%">商品名称</th><th width="10%">数量</th><th width="10%">金额（元）</th></tr>';
		template += '<% for(var i = 0; i < printdata.goodslist.length; i++) { %>';
		template += '<% var item = printdata.goodslist[i] %>';
		template += '<tr><td width="10%"><%=item.no%></td><td width="70%"><%=item.name%> </td><td width="10%"><%=item.count%></td><td width="10%">￥<%=item.amount%></td></tr>';
		template += '<% } %></table></div>';
		template += '<div class="totolAmount">订单总额：<span>￥<%=printdata.totalamount%></span>&nbsp; 元</div></div>';
		return template;
}
```







