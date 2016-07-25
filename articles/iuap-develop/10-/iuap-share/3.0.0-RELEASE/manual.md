# 互联网分享概述 #

## 功能简介 ##

提供分享当前页面到各大门户的插件。  

## 功能说明 ##   
1. 支持分享腾讯微博。
2. 支持分享新浪微博。
3. 支持分享QQ。
4. 支持分享QQ空间。
5. 支持分享百度贴吧。
6. 支持分享豆瓣。
7. 支持分享有道。
8. 支持分享搜狐微博。
9. 支持分享微信。     

*注：部分网站需要开通对应服务权限*  


# 整体设计 #


实质上组件提供的是前端JS 的各种API，不涉及到后台的调用。   
在*share.js*文件中，提供了分享到各大网站的调用方法`ShareToXXX()`。所以，只需要在前台页面中添加触发事件即可调用。

# 使用说明 #

## API接口 ##

    //分享到新浪微博
    function ShareToSina(title, pageurl, source) {
    window.open('http://v.t.sina.com.cn/share/share.php?title=' + encodeURIComponent(title) + '&url=' + encodeURIComponent(pageurl) + '&source=' + encodeURIComponent(source), '_blank');
    }
    
    //分享到QQ好友
    function ShareToQQIM(title, pageurl, source) {
    window.open('http://connect.qq.com/widget/shareqq/index.html?url=' +  encodeURIComponent(pageurl) +'&desc=&title= '+ encodeURIComponent(title) + '&summary=&pics=&flash=&site=&style=101&width=96&height=24&showcount=', '_blank');
    }
    
    //分享到腾讯微博
    function ShareToTencent(title, pageurl, source) {
    window.open('http://v.t.qq.com/share/share.php?title=' + encodeURIComponent(title) + '&url=' + encodeURIComponent(pageurl) + '&source=' + encodeURIComponent(source), '_blank');
    }
    
    //分享到百度贴吧
    function ShareToBaiduTieba(title, pageurl, source) {
    window.open('http://tieba.baidu.com/i/sys/share?title=' + encodeURIComponent(title) + '&link=' + encodeURIComponent(pageurl) + '&source=' + encodeURIComponent(source), '_blank');
    }
    
    //分享到豆瓣网
    function ShareToDouban(title, pageurl, source) {
    window.open('http://www.douban.com/recommend/?title='+ encodeURIComponent(title) + '&url=' + encodeURIComponent(pageurl) + '&source=' + encodeURIComponent(source), '_blank');
    }
    
    //分享到QQ空间
    function ShareToQzone(title, pageurl, source) {
    window.open('http://sns.qzone.qq.com/cgi-bin/qzshare/cgi_qzshare_onekey?url=' + encodeURIComponent(pageurl)+'&title=' + encodeURIComponent(title) , '_blank');
    }
    
    //分享到微信朋友圈
    function ShareToWeChart() {
    	
    	var qrcodeDIV = document.getElementById("qrcode");
    	if(qrcodeDIV.getElementsByTagName("div").length == 0){//需要生成二维码
    		
    		var pageurl = location.href;
    		
    		var qr = qrcode(10, 'Q');
    		qr.addData(pageurl);
    		qr.make();
    		
    		var qrcodeImgdom = document.createElement('DIV');
    		qrcodeImgdom.innerHTML = qr.createImgTag();
    		
    		qrcodeDIV.appendChild(qrcodeImgdom);
    	}
    
    	qrcodeDIV.style.display='';
    }

## 开发步骤 ##

1. 引入分享组件：  
在前台页面引入以下JS文件： 
 
		<script src="resources/js/share.js"></script>
		<script src="resources/js/qrcode.js"></script>
*注意：使用分享到微信朋友圈功能时，才需要在页面加入qrcode.js文件*
2. 在前台页面中添加分享按钮  

    	<a style="font-size:18px" onclick="document.getElementById('light').style.display='block'">分享: </a>  
3. 添加分享层DIV和各分享超链接  
此处仅以分享到腾讯微博和分享到微信为例：
	
		<!--分享层 -->
		<div id="light" style="display: none; position: absolute; top: 100px; left: 30%; width: 40%; height: 350px; padding: 16px; border: 5px solid gray; background-color: white; z-index: 1002; overflow: auto;">
			分享到   :
			<a href="javascript:void(0)" onclick="document.getElementById('light').style.display='none'"> 
		        <img src="resources/images/shareimg/x.png" border="0"  style="float:right;" alt="关闭" >
		    </a>
			<ul id="bsLogoList">
				<div id="bp-qqmb" style="float:left; width:50px;margin:15px;">
					<a href="javascript:void(0)" onclick="ShareToTencent(document.title,location.href,document.title);" class="tmblog"><img src="resources//images/shareimg/qqweibo.gif" border="0" alt="腾讯微博" ></a>
					<div style="font-size:10px;" title="腾讯微博">
						腾讯微博
					</div>
				</div>
				
				<div id="bp-wechart" style="float: left; width: 50px; margin: 15px;">
					<a href="javascript:void(0)" onclick="ShareToWeChart();" class='tmblog'>
						<img src="resources/images/shareimg/wechart.png" border="0" alt="微信分享">
					</a>
					<div style="font-size: 10px;" title="微信分享">微信</div>
				</div>
			</ul>
		</div>
		<!-- 二维码展示层 -->
		<div id="qrcode" style="display: none; position: absolute; top: 300px; left: 40%;  padding: 16px; border: 2px solid gray; background-color: white; z-index: 1003; overflow: auto;">
			<a href="javascript:void(0)" onclick="document.getElementById('qrcode').style.display='none';document.getElementById('fade').style.display='none'">
				<img src="resources/images/shareimg/x.png" border="0" style="float: right;"alt="关闭">
			</a>
		</div>
		<!--end分享层 -->

## 扩展机制 ##

用户可以自定义分享页面的UI，或者添加分享到其他SNS门户的按钮。

