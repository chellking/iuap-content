# Maven配置 #

在DevTool\repository\Maven目录下的本地仓库中，已经包含了iUAP平台基础组件，IDE中默认设置的Maven组件也是此目录，如果用户的开发工具包不在D盘的根目录，需要自行配置iUAP_STUDIO的Maven，配置方式如下如：
打开->窗口->首选项，选择Maven->Installations，确认安装的Maven为上述目录中的Maven。

![图](/articles/iuap-develop/4-/img/image003.jpg)

设置User Setting下的配置文件为开发工具目录下的maven的conf目录下的配置文件，如下图：

 ![工具目录]/articles/iuap-develop/4-/img/image004.jpg)
 
注意：如果开发工具包用户没有配置到D盘的根目录下，需要修改对应Maven下的conf目录下的settings.xml文件；

修改localRepository指定到自身的开发盘的对应目录下，示例：
```<localRepository>D:\DevTool\repository\Maven\Maven3.2.2\local\repo</localRepository>```
 
 IDE中的Maven默认设置了offline更新，如果用户配置了其他的Maven仓库，请根据自身情况，选择在线更新还是离线更新。


