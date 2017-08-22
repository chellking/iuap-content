# 前后端交互

示例默认采用Spring MVC的方式以Rest格式的http服务来向前端提供基础的数据访问能力，Spring MVC的配置，请参考`web.xml`和`spring-mvc.xml`。  
服务的声明如下：  


![](/articles/iuap-develop/4-/img/image025.jpg)


获取数据详细信息的示例如下图，前端通过ajax的方式获取后端数据，渲染页面。  

![](/articles/iuap-develop/4-/img/image026.jpg)


后端开发可以对上述的数据库操作类或者是服务类和Controller类进行单元测试，也可以采用浏览器访问的方式，调用GET类型的Rest服务进行测试。  

![](/articles/iuap-develop/4-/img/image027.jpg)
 