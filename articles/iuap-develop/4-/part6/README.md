# 调试运行

>  如果你的应用需要连接数据库，那么请先保证配置的数据库已经启动。如：运行bin目录下startPgsql.bat启动自带数据库。 

示例推荐使用的jetty方式进行调试运行，调试方式和上文中介绍的相似，调试过程中需要制定端口，上下文名称等，此类信息可以在`pom.xml`文件中进行配置。 

![jetty运行配置](/articles/iuap-develop/4-/img/image108.jpg)

上图是配置jetty运行的界面，本功能是有Maven提供实现，需要pom.xml配置插件实现，详细内容请查阅Maven插件。
