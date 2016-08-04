# 安装使用

> 参考介质说明解压安装盘，本节说明安装盘的初始化使用


# 安装向导

1.安装盘下载地址请看“[下载页面](#/platform3/articles/iuap-develop/2-/mian_fei_shi_yong.html)”
 
2.将下载下来的安装盘解压(建议解压到D盘根目录).

3.启动DevTool

```
    1.首次运行，需要执行\DevTool\initDevTool.bat
 
    2.运行DevTool\startDevTool.bat
```
## 升级

1. 工具本身不支持升级

2. 支持iuap2.01的项目工程直接导入到工具中即可.

# 产品卸载

1.	备份workspace.
2.	直接将DevTool文件夹删除即可.

> 注意:删除之前请注意备份好workspace.


# 商业授权使用 #

产品默认6个月的试用期，可以通过商业授权来申请商业license,这样产品就成为商业版本，不受试用期限制

具体license的安装请参考  [license服务器安装指南](#/platform3/articles/iuap-develop/9-/licenseserver/3.0.0-RELEASE/manual.html) 

# 注意事项 #

开发环境解压完成就可以使用与启动开发工具，但是每个开发人员放置开发目录放的位置不同，需要根据不同目录配置开发工具的相应用Maven路径等，"initDevTool.bat"就是写好了一个初始化默认位置配置的批处理工具。

首次运行指导:

1. 运行开发环境前请先右键以管理员身份运行开发根目录下的initDevTool.bat。           
2. 强烈建议解压到D盘根目录，如果没有处在D盘根目录，请运行DevTool目录下的startDevTool.bat开启开发环境IDE，自行设置maven环境到DevTool下的maven库, 设置方法: 在studio中点击：窗口 > 首选项 > Maven > User Settings。修改Global Settings 和User Settings为当前devtool中的\repository\Maven\Maven3.2.2\conf\settings.xml文件。点击  Update Settings 按钮,更新Local Repository。     
3. 运行bin目录下的startpgsql.bat，并在开发环境的IDE中以jetty的方式调试示例工程。
4. 如果打开postgresql数据库发生闪退，需要对DevTool\DB\pgsql文件夹赋予完全控制权限。对文件夹点击右键选择属性--选择安全标签--点击编辑--为当前用户添加完全控制权限。
5. 运行postgresql数据库需要安装vc2010运行库，DevTool\DB\pgsql文件夹下vcredist_x86.exe为安装包。

升级提示:  

1. 解压后放到d盘根目录，若之前运行过老版本的本开发环境，删除时候注意备份对应的workspace下的工程，以免误删除！
2. 建议运行环境win7 64位 4g内存 具有vc2010运行库。

开发模式：

按照需求启动bin目录下的startpgsql.bat、startredis.bat、startsolr.bat、startzookeeper.bat 完成开发启动准备，启动根目录下的startDevTool.bat可以打开默认的开发工具，examples下内置了工程的示例代码，开发人员可以直接调试和查看代码，根据官网文档的指导快速介入。
```
1.“startDevTool.bat”说明：开发工具启动入口；

2.此类启动方式仅供快速浏览和开发模式熟悉，正式开发环境搭建请参考工具及规范；

3.批处理启动时候使用了8080和5432端口，请保证对应端口未被占用，对应的服务可以启动。
```
