# 前端教程

有没有想过在实际开发中，不会因为用户的操作而进行页面的重新加载或跳转，取而代之的是动态的变换HTML的内容，避免页面的重新加载，这种用户体验是不是很好。所以开发平台前端开发中，默认采用了AMD模块化规范的单页面应用(SPA)。在页面中，采用了iuap Design的[UI控件库](http://design.yyuap.com/dist/pages/neoui/index.html)以及[模型框架库](http://design.yyuap.com/dist/pages/kero/overview.html)分别来做UI层开发和数型层(MVVM)开发。

下面我们聊聊上面所提到的前端页面开发、后端数据交互、AMD模块化、单页面应用以及搭建项目所用到的技术。

## 前端页面开发

前端页面开发使用了iuap Design的UI框架以及数据模型框架，iuap Design 是企业级Web应用开发前端整体解决方案。包含设计语言、前端框架、插件库、组件库、模板库等。

平台中使用了iuap Design中的控件库做为默认控件，同时不限制开发者引用第三方控件库，如做图表开发时，我们会引用echart.js 。

平台页面中，采用了iuap Design的模型框架,一种基于knockout.js扩展的MVVM模型做为主开发框架。

主要优势有：
- 完善的控件体系：包含所有常见的控件。
- 兼容IE8以上的主要浏览器：IE 8+、Firefox、Chrome、safari
- 多端适配 ： 使用响应式的CSS在电脑、手机和平板电脑上看起来都很好看
- 声明式绑定：使用简明易读的语法很容易地将模型数据关联到DOM元素上
- 双向数据绑定：模型与UI之间的双向自动更新
- 多维数据模型：解决了字段关联、主子数据、主子孙等多维数据模型的绑定问题。


页面开发中的UI相关问题，可访问地址： [http://design.yyuap.com/dist/pages/neoui/index.html](http://design.yyuap.com/dist/pages/neoui/index.html) 进行学习。


页面开发中的数据模型相关问题，可访问地址： [http://design.yyuap.com/dist/pages/kero/overview.html](http://design.yyuap.com/dist/pages/kero/overview.html) 进行学习。

前端页面搭建完毕后，那如何与后端进行数据交互？

## 与后端的数据交互

前端框架与服务端没有耦合，中间通过JSON数据格式进行数据传递。可采用正常的ajax请求。例如，使用jquery的ajax请求：
```
    $.ajax({
        url:'api/xxx',
        data:{xxx:xxx},
        success:function(data){
            alert(data.msg)
        }
    })

```

如果要发送数据模型中的相关数据，需要把数据模型中的数据转成JSON发请到服务端，参考iuap Design中的数据模型相关API用法：

[http://design.yyuap.com/dist/pages/kero/udatatable.html](http://design.yyuap.com/dist/pages/kero/udatatable.html)


前端页面和后端交互完成后，让我们来看看多个页面如何关联的，先来了解下AMD模块化的单页面应用(SPA)。

## AMD模块化规范


AMD是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义"。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

AMD采用require()语句加载模块，但是不同于CommonJS，它要求两个参数：

```
require([module], callback);
```

第一个参数[module]，是一个数组，里面的成员就是要加载的模块；第二个参数callback，则是加载成功之后的回调函数。例如:

```
　　require(['math'], function (math) {
　　　　math.add(2, 3);
　　});
　　
```

math.add()与math模块加载不是同步的，加载完math模块，运用math的add方法，这样浏览器不会发生假死现象。所以很显然，AMD比较适合浏览器环境。

目前，require.js是主要是实现了AMD规范的javascript库。平台中也是使用的require.js做为AMD模块化规范。可参见：http://www.requirejs.cn/ 学习requirejs的用法。


## 单页面应用(SPA)

单页Web应用（single page web application，SPA），就是只有一张Web页面的应用。单页应用程序 (SPA) 是加载单个HTML 页面并在用户与应用程序交互时动态更新该页面的Web应用程序。 浏览器一开始会加载必需的HTML、CSS和JavaScript，所有的操作都在这张页面上完成，都由JavaScript来控制。因此，对单页应用来说模块化的开发和设计显得相当重要。

单面面应用的优点是：
- 速度：更好的用户体验，让用户在web app感受native app的速度和流畅，
- MVC：经典MVC开发模式，前后端各负其责。
- ajax：重前端，业务逻辑全部在本地操作，数据都需要通过AJAX同步、提交。
- 路由：在URL中采用#号来作为当前视图的地址,改变#号后的参数，页面并不会重载

在平台中,实际只有index.html这一个单页面，在这个单页面中，加载了必须的CSS和JavaScript。然后有一个内容区，在点击菜单时，改变地址hash，我们用hash的变化从而推动界面相应的功能变化并在内容区中渲染出来。

那如何根据hash变化调用所需的js方法呢？来让前端路由给我们答案吧！

## 前端路由

前端路由主要是用在单页面应用中，解决单页面应用里，地址变化，整体页面并不需要重新渲染，只重新渲染局内容。

平台中采用[director.js](https://github.com/flatiron/director)开源框架做为默认的路由框架。基本用法如下：

```
var router = Router();
//定义order路径所对应的操作
router.on('/order', function(){
    // 具体的实现代码
});

router.init();

```

当浏览器地址变为：`http://localhost#/order`时,会触发上面监听的function。

在平台前端主页面中，使用了router的相关方法来监听地址变化。同时使用requirejs来做动态加载。从而实现的整个单页面应用的运转。

##总结

以上概述了一个前端工程构建的基本流程，从单页面编写、后端数据交互交互、到多页面的关联。平时开发者还可以积累一些项目脚手架，来提高开发效率。[iuap design](http://design.yyuap.com/)提供了一些脚手架例如[cloud-starter-kit](https://github.com/iuap-design/cloud-starter-kit).开发者以此为参考，快速构建项目。