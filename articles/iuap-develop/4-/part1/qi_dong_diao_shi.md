# 启动调试

> 运行bin目录下的startPgsql.bat，启动数据库。  

示例工程中配置好了Jetty这个WEB容器，支持使用jetty调试，右键工程名，选择调试方试->Maven build…  

![](/articles/iuap-develop/4-/img/image005.jpg)
 

在右侧上方的页签中选择[源]，添加源代码，选择需要添加的工程，加入到调试工程中，以方便断点调试。  

![](/articles/iuap-develop/4-/img/image006.jpg)

在Main页签中的[goals]中，输入"jetty:run"，选择跳过测试"Skip Tests"，点击应用，调试，启动工程的jetty调试。  


![](/articles/iuap-develop/4-/img/image007.jpg)


正常情况下，启动调试完成后，在控制台中可以看到如下信息：   


![](/articles/iuap-develop/4-/img/image008.jpg)


在浏览器中输入[<http://localhost:8080/iuap-quickstart>](http://localhost:8080/iuap-quickstart)，访问示例应用，界面如下：  


![](/articles/iuap-develop/4-/img/image009.jpg)
 

示例为商品基本的(CRUD)增删改查功能，开发人员可以在工程中查找到对应的源码，在其它功能的开发中参考。  
