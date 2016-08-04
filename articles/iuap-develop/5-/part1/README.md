# 工具说明

本章节主要是对开发工具的说明，在开发前先要熟悉开发工具能为我们做什么，这样我们在使用中才会获得工具提供的帮助，实现快速开发

##iuap Studio开发工具概述与关键特性

### 开发工具概述

iuap Studio是一个简单易用的基于Eclipse开发平台的web项目开发工具。工具集成了eclipse j2ee版本的大部分插件，大大降低了学习成本，并基于iuap开发平台量身定做了很多实用功能，为WEB应用开发者提供便捷的可视化建模开发，包括快速生成代码块、提供界面友好的可扩展可定制的代码提示窗口、html标签扩展、JS第三方语法支持、页面模式和向导、组件仓库的管理、快速代码生成引擎以及元数据模型设计发布等功能。

### 代码助手，极速编程体验

#### 个性化编辑器:可拖拽的控件工具箱

web页面编辑器左侧提供控件工具箱，点击小箭头可以收起/展开

![](./image/studio_1.png)

Studio预置了一些html基础控件以及iuap定制控件，可通过双击或者拖拽的方式插入html页面代码中。 
用户可以也根据需要自定义控件，并使用提供的导入/导出功能完成控件的分享。

![](./image/studio_2.png)

#### 代码智能提示：更快更炫更全

不用输全代码，系统会根据已输入的内容智能弹出提示框，辅助用户进行代码补全。 在补全列表中，可直接使用数字键等快捷键直接选中目标代码。
![](./image/studio_3.png)
studio不仅仅提示全面的语法，非语法的各种候选输入也都能提示。包括图片、链接、颜色、字体、脚本、样式、URI、ID、class、自定义JS对象、方法。
举例，在img src=后激活代码助手，可以看到本工程所有图片列表；输入<sc可以看到本工程所有js列表；在js的document.getElementById(id)中提示本工程已经定义的ID列表；在css的color:后可以列出本工程所有使用过的颜色。

![](./image/studio_4.png)

#### 自定义代码块：解放双手，告别重复劳动

可重复利用的代码块可以使用提供的代码块管理窗口录入。提供触发器和代码片段后，即可在智能提示框中使用。

![](./image/studio_5.png)
![](./image/studio_6.png)

#### JS第三方框架语法支持：前端开发利器

内嵌了jquery、bootstrap、angular、mui等常用框架的语法提示库，并且这些框架语法一样可以享受到智能提示机制，提示图片、颜色、id、class...
如果要使用框架语法，需要在工具菜单/项目的右键菜单中，点击引入框架语法提示的子项，为该项目选择框架语法提示。提示效果如下

![](./image/studio_7.png)

### 模版引擎，没有什么是不能模式化的

Studio提供了丰富的模版内容，这些内容靠模版引擎驱动，除了预置模版之外还支持用户自定义模版内容，以内容来分大概分为工程模版，页面模版和代码模版三类。

#### iuap工程

studio内置iuap示例工程，通过可通过右键工程或者新建iuap工程向导完成。 如图所示：

![](./image/studio_8.png)
点击完成后， 工作空间内会生成包含iuap平台基础框架和控件的示例工程，可以直接启动tomcat运行，可以快速预览页面内容。

#### iuap页面

studio内置iuap标准页面，通过新建iuap页面向导完成。 可通过右键工程或者新建向导创建。如图所示：
![](./image/studio_9.png)

新建iuap页面有两种类型，一个是标准的不使用数据模型的空页面，一个是使用数据模型的，可以关联iuap DataTable的数据页面。选择使用数据模型，点击下一步如下：

![](./image/studio_10.png)

点击完成后会生成html和js类型的两个文件。 用户可在html的div中补充页面内容。 js描述了html页面的动作和数据关联。

#### 页面模版

按领域和功能模块提供了丰富的页面模版。主要是前端页面，包含样式，脚本以及html内容。
根据领域选择中意的页面样式，点击完成即可生成包含完整样式的静态页面。 

![](./image/studio_11.png)

#### 页面预览

使用新建页面完成页面内容或者使用页面模版生成页面后，启动web服务器，右键工程选择页面预览或者点击工具栏中页面预览按钮，可以直接在浏览区预览页面效果。

#### 代码模版

通过选择数据源，勾选需要的实体属性，可以方便的生成基于数据源的后端代码， 预置的模版提供了jpa以及iuap－jdbc方式生成javabean以及DAO的功能。

### 元数据设计器, 最有效率的模型设计工具

模型设计器作为实体模型、数据库模型的定义以及代码生成和脚本生成的工具，为业务系统中的领域模型的建立和基础代码的产生提供了一个完整的平台。

####元数据管理器

元数据管理器是提供元数据的相关管理工具，它支持元数据的创建、数据库导入实体组件、导入元数据bmf文件及删除元数据文件同时方便查看所有模型资源以及公共资源。

![](./image/studio_12.png)

####实体操作

对元数据实体的操作可以很直观的在元数据编辑器(画布)上反映出来。 实体组件工具面板上用户通过拖拽的方式，在画布上创建实体组件、创建枚举、创建业务接口、创建注释以及创建关联关系。

![](./image/studio_13.png)
右键点击实体模型弹出菜单，可执行元数据发布，脚本生成，导出代码等实用操作。

### 组件仓库，提供iuap组件化管理

####组件云商店

提供直连通道连接云端组件库， 直观体现给用户iuap平台提供的所有组件，并提供组件的下载安装。

![](./image/studio_14.png)

####本地工程组件管理

提供工程相关的组件配置方式，方便用户使用本地组件库中组件。 

![](./image/studio_15.png)


# 功能说明

## iuap Studio 基本功能说明与应用

### 1. 新建向导

在左侧项目管理器中点击右键，选择新建->项目，进入新建项目向导。

#### 1.1 新建Java工程

向导中选择Java->Java项目，在新窗口中创建你的工程名字，选择你所用的jre环境（这个取决于你电脑上安装的JDK版本），然后点击完成，即可创建普通java工程。 

#### 1.2 新建Spring工程

向导中选择Spring->Spring Project，在新窗口中创建你的工程名，选择相应的工程模版，点击完成，即可创建Spring工程。

#### 1.3 新建Maven工程

向导中选择Maven->Maven Project，在新窗口中选择是否新建普通maven工程(create a simple project)，选择则下一步直接填写groupid及artifactid等，点击完成即可创建maven工程，填写的信息则包含在创建工程的pom文件中。 不选择新建普通maven工程，则下一步可以选择系统提供的maven模版，点击完成新建定制maven工程。

#### 1.4 新建iuap工程

向导中选择iuap->iuap项目或者直接在左侧项目管理器中点击右键，选择新建->iuap项目，在新窗口中点击名称和ArifactId(工程id)，点击完成，即可创建基于iuap示例工程模版的iuap工程。工程中带有若干示例页面和完整的组件信息以及前端uui组件，可以直接启动并在浏览器中预览示例工程页面。

#### 1.5 新建iuap页面

在左侧项目管理器中选择已有的iuap工程， 点击右键，选择新建->iuap页面，新窗口中填写页面名称和id，并选择是否使用数据模型，点击完成即可新建iuap页面（仅在iuap工程中使用）,iuap页面包含html和js文件。

#### 1.6 新建java类(实体类)

在左侧项目管理器中选择已有工程， 点击右键，选择新建->类，建立空白java类，或者新建->JPA Entities from Table，可以根据数据表新建基于数据库字段的javaBean。

### 2. 工程导入

在左侧项目管理器中点击右键，选择导入->导入，进入导入向导。

#### 2.1 已存在的工程导入

此功能提供将已有的eclipse工程导入工作空间，在导入向导中选择导入已有工程，选中要导入的工程目录，在列表框中选择要导入的工程名，点击完成即可。 

#### 2.2 Maven工程导入

此功能提供将已有的maven工程导入工作空间。 在导入向导中选择导入已有maven工程，选中要导入的工程目录，在列表框中选择要导入的工程pom文件，点击完成即可导入工作空间并自动完成编译构建。

### 3. JRE环境配置

工程的编译执行运行时需要JRE的支持，因为Studio安装目录里是不带JRE的，因此要使用Studio开发首先就要安装好JRE或JDK，然后在Studio中配置jre运行时环境。在主菜单中选择窗口->首选项，里面打开Java->Installed JREs，在右侧页面中配置工作空间需要的java运行时环境。

### 4. Maven仓库支持

目前使用maven作为项目管理工具开发的越来越多。Studio支持提供对maven的支持。在使用maven的时候，首先要对工作空间的maven进行相应的配置才能够正常使用并且创建maven工程。

#### 4.1 Maven环境配置

maven安装目录下有个config文件夹，在文件夹下有settings.xml文件。settings里面就是配置maven工程的一些本地仓库地址（项目使用的某些jar下载并且安装到本机中），远程仓库（如果本机没有需要的jar，则从远程获取。）的配置等信息。在主菜单中选择窗口->首选项，打开Maven->User Settings配置页， 可以配置所用的maven的settings文件位置，并列出本地仓库地址。

#### 4.2 Maven工程pom配置

打开工作空间中已有的maven工程或iuap工程，双击pom文件，可以打开pom编辑器，在里面可以配置maven工程的groupid,artifactid,版本，打包方式，还有依赖组件等。

#### 4.3 Maven更新

使用pom编辑器更改maven工程信息后，右键点击maven工程，选择Maven->update project，可以按照最新配置更新maven工程，添加新的依赖组件等等。

#### 4.4 Maven构建和启动服务

右键点击maven工程，选择运行方式，子菜单中列出了多种maven的执行命令。其中选择Maven build，在新窗口的Goals文本框中可以自主填写maven命令。如tomcat:run，或者jetty:run，启动tomcat或者jetty服务器。 

### 5.Spring开发套件

#### 5.1 Spring工程模版

新建向导中选择spring project，下一步可选择多种spring工程模版，如spring maven，spring web，spring mvc等。

#### 5.2 Spring工程资源树

建立spring工程后， 左侧项目管理器中显示新建的spring工程，展开工程树结构，显示部署描述符，Spring Elements，JPA content等多种功能节点。 方便对spring工程进行管理。

### 6.Git版本管理支持

#### 6.1 git环境配置

依次点击菜单窗口——首选项——小组——Git——Configuration，点击其中的Add Entry按钮添加用户信息。
这里的Key-Value键值对是提交代码时的备注信息，这里我们可以添加用户名和邮箱等信息。
其中Key中输入User.name, 然后Value中输入用户名，点击OK；这样就将User用户的用户名添加成功了。然后再次点击Add Entry按钮，添加User用户的Email信息，其中Key中写下user.email，Key中输入你的邮箱名，点击OK，此时user用户的邮箱信息即添加完成。当然，你也可以添加其他的备注信息。接下来我们来配置我们的本地Repository。在Eclipse中选中我们要备份的项目，然后依次点击右键——Team——Share Project…此时将打开Configure Git Repository对话框。选中我们要备份的项目，然后点击Create…按钮，选择我们项目Repository的本地存储路径，即本地仓库。点击Finish。此时我们的本地仓库已建立完成。

#### 6.2 从git服务器导出git项目

在项目管理器中，右键选择导入，在导入对话框中，选择“Git”，然后再选择“Projects from Git”。在在弹出的“Import Projects from Git”弹出框中输入“URI”、“Host”、“Repository path”、“User”、“Password”输入框的值，再选择分支和存储路径，点击完成，项目即可在工作空间中加载。

#### 6.3 代码的提交与更新

如果工作空间中有git工程， 打开Git Respositories视图,在此git管理视图中右键点击工程，可选择执行git工程的commit,pull,push等操作。

### 7.JPA持久化规范支持

JPA 充分利用了注释（Annotation）和对象 / 关系映射，为数据持久化提供了更简单、易用的编程方式。Studio中提供了对JPA的支持。

#### 7.1 JPA配置

将已有项目添加JPA支持，首先选择项目右键Properties->Project Facets 勾选列表Project Facts 下JPA选项，点击Furhter configuration available，转入数据库连接配置。数据库配置中选择相应类型的数据库和驱动文件，完成主机ip，用户名，密码等信息的录入后，即可完成配置。

#### 7.2 JPA持久化

配置完成后，在项目文件夹上右键选择JPA tools->Generate Entities from Tables，可以执行从数据库创建java实体类或者选择Generate Tables from Entities，执行将javaBean发布到数据库表中。 

### 8.实用编辑器

#### 8.1 java编辑器

双击工程中的java类型文件，可以进入java编辑器。java编辑器提供了代码提示，断点设置，代码重构，打开类层次结构和调用层级结构，抽取方法，格式化元素等多种功能。 

#### 8.2 html编辑器

双击工程中的html，html等类型文件，进入html页面编辑器。html页面编辑器提供了智能代码提示，代码块补全，控件工具箱，html标签扩展，代码块模版扩展等多种功能。

#### 8.3 js编辑器

双击工程中的js文件，进入js编辑器。js编辑器提供智能代码提示，代码块补全，第三方js框架语法引入，代码块模版扩展等功能。

#### 8.4 css编辑器

双击工程中的css文件，进入css编辑器。 css编辑器提供智能代码提示，多种css样式引入，颜色板提示等功能。 

#### 8.5 xml编辑器

双击工程中的xml文件，进入xml编辑器。xml编辑器提供智能代码提示，xml设计等功能。并有xml架构文件编辑器，提供xml文件的模式化生成和实体类的生成。

### 9.服务器支持

Studio中提供对多种服务器的支持。以Tomcat服务器为例。

#### 9.1 服务器配置

点击窗口->首选项->服务器->运行时环境，进入服务器配置页。点击右侧添加按钮，进入服务器选择页。进行若干配置，即可完成。

#### 9.2 服务器视图

点击窗口->显示视图，在视图选择页中选择服务器视图。 在服务器视图中，可以完成服务器的启动和停止。

### 10.数据库支持

#### 10.1 数据库视图

点击窗口->显示视图,选择Data Management->Data Source Explorer, 打开数据库配置视图。 在配置区选择Database Connections，右键New，在新窗口中选择数据库类型和数据库驱动等，并完成主机ip，用户名密码等信息的配置即可。 完成配置后，在数据库视图，可以显示数据库中所有的表信息并提供对数据库的操作。

#### 10.2 工程相关数据库配置

Studio为iuap工程提供了专门的数据源配置页。 右键iuap工程，选择iuap tools->数据源配置，在打开的数据源配置对话框中，输入数据库相关信息，并测试连接通过，即可。通过数据源配置页设置的数据源信息保存在application.properties中。

#### 10.3 数据库应用

数据库配置有两块，通过数据库视图中配置的数据库信息是适用于整个工作空间的，在JPA工具进行持久化操作的时候使用。通过iuap tools设置的数据源信息用于iuap工程的启动，浏览和调试以及元数据发布等。

### 11.数据模型引入

开发工具中的数据模型有很多用途，比如绑定控件替换参数，一键生成数据绑定配置文件，页面模式化向导等。 这里的数据模型的概念实际上是基于JavaBean实体类的xml文件，以.model为后缀名。

#### 11.1 新建数据模型

iuap开发工具提供两种方式生成数据模型。
一是通过数据模型向导直接新建模型。右键工程，选择新建->数据模型。下一步输入模型的名称以及位置，点击完成后，通过model编辑器完成模型的录入。
二是通过绑定实体类，自动生成数据模型。 初始步骤同上，在输入模型名称后点击下一步，进入属性配置页面。 选择需要的实体类以及属性后，点击完成即可生成与实体类对应的模型文件。

#### 11.2 数据模型的使用

数据模型可以用来生成数据控件和生成数据页面。
在html编辑器控件工具箱中拖放数据表格等控件，可以绑定数据模型，生成基于数据模型的代码块
在新建iuap页面功能中，点击使用数据模型，可以绑定数据模型，生成基于数据模型的iuap页面。

### 12.页面模版与代码模版

#### 12.1 根据模版新建页面

点击工具栏新建->iuap页面模版,可以使用页面模版创建页面。 Studio初始安装后， 预置了通用和电商两个领域类别的模版，包含了近30套页面。用户可以根据场景选择模版，点击下一步，输入页面名称即可完成，生成的页面默认放在工程下的src/main/webapp/pages目录中。
用户也可以根据已有页面生成可用模版。在pages下已完成的页面目录点击右键，选择iuap Tools->生成页面模版，进入页面模版生成向导。 填写完成名称，分类等信息后，点击完成模版会生成在指定位置。这时，可以在新建->iuap页面模版中可以看到刚导出的页面模版了。

#### 12.2 快速代码生成向导

代码片段通过代码块模版生成，前端页面通过页面模版生成，后端代码则可以通过快速代码生成实现。 右键点击工程，选择iuap Tools->快速代码生成， 进入代码生成向导。 首先选择数据源，这里有两种方式， 从元数据导入和从数据库导入。

### 13.元数据设计器

#### 13.1 元数据管理视图

元数据管理器是提供元数据的相关管理工具，它支持元数据的创建、数据库导入实体组件、导入元数据bmf文件及删除元数据文件同时方便查看所有模型资源以及公共资源。

#### 13.2 元数据编辑器

模型编辑器器是一个图形化的工具，画布工具便于使用者进行类图建模的绘制。使用者从工具面板中拖动相应元素到画布类图中创建实体、枚举、值对象等。并可对类图中的对象进行直接增、删、改操作。

#### 13.3 元数据的发布和生成建库脚本

元数据编辑器中选定类图中对象后点击鼠标右键可以进行导入属性、发布元数据、发布元数据（忽略版本）、删除发布元数据、生成java源代码、导出pdm(.xml)文件等功能。也可以在元数据管理器中右键点击元数据文件，实现上述功能。

### 14. 组件管理

iuap平台为组件化管理。 把基本功能按照组件拆分。通过maven仓库管理组件。Studio通过可视化界面， 提供maven组件的下载安装管理

#### 14.1 组件仓库

点击主菜单栏的工具->组件仓库，打开“组件商店”。连接官网提供的组件仓库，按分类显示所有组件的详情描述，并提供下载安装按钮

#### 14.2 工程组件管理

右键点击工程，选择iuap Tools->工程组件。 显示本地所有已安装的iuap组件。 并通过勾选组件决定工程是否使用本地组件。 

### 15.中间件启动和页面预览

通过服务器视图，选择已经配置的服务器，可以完成服务器的启动，或者通过右键工程选择运行方式->maven build，在Goals中填写tomcat:run或者jetty:run,启动tomcat或者jetty服务器。启动完成后，如果是iuap工程，还可以实现页面预览功能。 右键点击pages目录下的page，选择iuap tools->页面预览，可以在浏览器中直接预览页面。 

### 16.搜索和替换

Studio提供对整个工作空间或者单个工程或文件的搜索和替换功能。点击主菜单的search->搜索菜单，进入功能对话框。

### 17.安装和更新

Studio提供新插件的安装以及Studio的自动更新功能。 点击主菜单帮助->安装新软件，在work with中输入插件在线更新地址，点击enter，然后选择插件，点击下一步进行安装，或者选择帮助->Eclipse Marketplace,打开插件商店，可以搜索需要的插件进行安装。
工具可以在启动时自动检查更新，或者点击帮助->检查更新手动执行Studio更新，我们会定期更新工具的最新功能和模版.

### 18.授权管理

此功能需要配合license服务器使用。在license服务器中导入授权后，点击主菜单帮助->授权管理，在窗口中填写服务器地址、端口，并选择产品进行授权。不授权可以免费使用6个月， 授权后在授权管理窗口可以看到授权有效期。

### 19.帮助内容

点击主菜单->帮助->帮助内容，可以查看Studio相关帮助信息。

### 20.欢迎界面

初次运行iuap Studio或者点击主菜单->帮助->欢迎，可进入开发工具欢迎界面。 点击相应按钮可进入开发工具说明及使用手册，也可以直接点击新建按钮进入新建iuap工程向导。



# 开发工具使用

# 1.iuap Studio 开发工具

## 1.1 产品概述

iuap Studio是一个简单易用的基于Eclipse开发平台的web项目开发工具。工具集成了eclipse j2ee版本的大部分插件，大大降低了学习成本，并基于iuap开发平台量身定做了很多实用功能，为WEB应用开发者提供便捷的可视化建模开发，包括快速生成代码块、提供界面友好的可扩展可定制的代码提示窗口、html标签扩展、JS第三方语法支持、页面模式和向导、组件仓库的管理、快速代码生成引擎以及元数据模型设计发布等功能。

## 1.2 iuap_quickstart实例工程

iuap_quickstart工程是iuap开发平台的实例工程，该工程基于iuap开发框架进行基本业务逻辑的编写,实现了完整的增删改查逻辑，并在具体的示例页面上加以展示。可以帮助业务开发人员快速的完成对平台基本功能的了解和使用。工程中包含了完整的iuap基础组件和前后端框架，开发人员在iuap Studio中可以通过新建iuap项目来创建，并在此基础上进行开发。

# 2.开发流程

## 2.1 环境配置

### 2.1.1 JDK配置

开发工具默认JDK位于安装目录下的Runtime目录。 在IDE中，选择窗口->首选项->JAVA->已安装的JRE，可以看到目前IDE中使用的JAVA环境。如果不使用默认的JDK，可手动添加本地JDK，勾选即可。

![](./image/Jdk.png)

### 2.1.2 Maven配置

开发环境的maven位于iuap开发工具包的DevTool\repository\Maven\Maven3.2.2目录下，也可以自定义使用本地maven。以默认maven为例，首先要设置maven的环境变量，在系统变量中添加环境变量MAVENHOME,该变量指向目录DevTool\repository\Maven\Maven3.2.2，然后在Path变量中加入路径%MAVENHOME%\bin。 maven环境变量配置好以后，我们要在IDE中配置maven的配置文件的路径，在IDE的菜单栏中选择窗口->首选项->Maven->User Settings，将标签页的Global Settings和User Settings改为开发工具包内maven的配置文件的地址  

![](./image/Mavensetting.png)

### 2.1.3 授权管理

iuap Studio初次使用可以免费使用6个月。在6个月免费期内无需注册即可享受全部功能。 6个月后，若不进行授权，则无法使用部分功能。

获取授权需要启动license server，具体过程请参考license服务器使用说明文档。

服务器获取授权后，保持联机状态，点击主菜单栏->帮助->授权管理， 进入授权管理窗口。 在文本框中输入license服务器的ip和端口，点击“ping”按钮，确认license server已联机。

根据需要，勾选工具产品后，点授权按钮。客户端将向License Server 发出注册申请。一旦服务端授权成功，客户端会弹出提示框获取授权。

![](./image/license.png)

已经获取授权的客户端允许在离线时限（30天）内，不用每次使用时都联接License Server进行认证。

超过有效期的客户端不能继续使用，此时在License Server端可以注销该用户。

## 2.2 工程初始化

### 2.2.1 新建iuap项目

在IDE菜单栏中选择新建->iuap项目，点击下一步，进入新项目的配置页面。 输入项目名称建立新项目 如图所示：  

![](./image/iwebproject.png)

![](./image/studio_8.png)

点击完成，稍等片刻，IDE在完成maven库的更新下载以及工程编译的进度读条后，这时工作空间中会建立一个新的iuap工程。

### 2.2.2 数据源配置

右键工程，选择iuap Tools->数据库连接配置。 在弹出的对话框中配置数据库类型和基本信息，点击测试连接，测试通过后，点击完成，即完成了对工程数据源的配置。

![](./image/database.png)

## 2.3 建模过程

### 2.3.1 实体类层编写

iuap开发工具可使用**三种方式**快速生成javabean实体类。

**方式一：** 工具使用JPA插件，通过数据表的方式自动建立实体类，首先建立好数据库、实体表、以及数据库连接，在工程上右键选择配置->转为JPA工程，完成转化后，右键工程选择JPA tools->Generate Entities from Tables, 从数据表反向生成Entity。  

![](./image/iwebJPA.png) 

![](./image/iwebJPA2.png)

**方式二：** 通过元数据设计器设计元数据模型，并生成java源代码。

![](./image/metajava.png)  

**方式三：** 通过代码快速生成向导，选择适用的模版类型生成相应格式的javaBean。

![](./image/codegenjava.png)  

选择生成字段以及包名等，工具自动生成注解，生成实体类如下：   

```
		@Entity

		@Table(name="example_demo")

		@NamedQuery(name="Demo.findAll", query="SELECT d FROM Demo d")

		public class Demo implements Serializable {

		    private static final long serialVersionUID = 1L
		    
		    @Id
		    private long id;

		    @NotBlank(message="测试编码不能为空!")
		    private String code;
		    
		    private String memo;

		    private String name;

		    private String isdefault;

		    public Demo() {

		    }

		    public long getId() {

		        return this.id;

		    }

		    public void setId(long id) {

		        this.id = id;

		    }

		    public String getCode() {

		        return this.code;

		    }

		    public void setCode(String code) {

		        this.code = code;

		    }

		    public String getMemo() {

		        return this.memo;

		    }

		    public void setMemo(String memo) {

		        this.memo = memo;

		    }	

		    public String getName() {

		        return this.name;

		    }

		    public String getIsdefault() {

		        return isdefault;

		    }
		
		    public void setIsdefault(String isdefault) {

		        this.isdefault = isdefault;

		    }	

		    public void setName(String name) {

		        this.name = name;

		    }

		}

``` 

### 2.3.2 数据模型引入

开发工具中的数据模型有很多用途，比如绑定控件替换参数，一键生成数据绑定配置文件，页面模式化向导等。 这里的数据模型的概念实际上是基于JavaBean实体类的xml文件，以.model为后缀名。iuap开发工具提供两种方式生成数据模型。   

一是通过数据模型向导直接新建模型。右键工程，选择新建->数据模型。下一步输入模型的名称以及位置，点击完成后，通过model编辑器完成模型的录入。 如图所示：   

![](./image/model1.png)   

![](./image/model2.png)  

![](./image/model3.png)   

![](./image/model4.png)   

二是通过绑定实体类，自动生成数据模型。 初始步骤同上，在输入模型名称后点击下一步，进入属性配置页面。 选择需要的实体类以及属性后，点击完成即可生成与实体类对应的模型文件。  

生成的模型文件如下所示：   

```

		<?xml version="1.0" encoding="UTF-8"?>

		<model xmlns="http://www.yonyou.com/uap/DataModel" className="uap.web.example.entity.Demo" id="Demo" name="Demo">

		  <attributes>

		    <attribute id="id" name="id" type="long"/>

		    <attribute id="code" name="code" type="String"/>

		    <attribute id="memo" name="memo" type="String"/>

		    <attribute id="name" name="name" type="String"/>

		    <attribute id="isdefault" name="isdefault" type="String"/>

		  </attributes>

		</model>

```

## 2.4 新建页面

右键点击工程根目录或者在工具栏中选择新建->iuap页面，进入iuap页面新建向导。 

![](./image/page1.png) 

页面统一建在工程下的src/main/webapp/pages目录下。 填写页面名称后，可以选择是否使用数据模型，如果不勾选数据模型，则可以点击完成，会创建仅包含基础js框架的页面。 若勾选，则点击下一步，进入数据模型绑定页。

![](./image/page2.png)

下拉框中选择上一步创建的数据模型，并勾选用到的属性。用户可以通过勾选“选择”按钮使用多个数据模型，点击完成即可创建使用数据模型的html和js页面。

![](./image/page3.png)   

## 2.5 数据绑定

在有些情况下，可能不是要创建整个功能页面，而是在页面中添加一些功能控件，完成数据操作。 比如上图中的列表控件和表单控件。如果要单独添加这些控件如何做呢，这就需要用到控件的数据绑定。我们这里提供了目前最常用的两种控件模板，表格和表单。

以表格为例，默认使用web页面编辑器打开html页面文件。在左侧工具箱中选择控件下的简单表格项，双击打开数据配置页，关联上面创建的数据模型，点击完成即可在html中插件相关代码，如图所示：   

![](./image/editor.png)   

![](./image/codeinsert.png)  

完成html代码创建后，并不意味着可以和后台数据做关联，数据还需要采用knockout的方式进行双向绑定，所以我们需要支持创建这种数据绑定js文件，

选中工程， 右键新建，在向导页中选择iuap->数据绑定向导。 下一步选择js文件的位置，这里js文件一般需要与相应的html文件放在同一级目录下并且命名一样。下一步选择数据模型及其属性，点击完成即可创建JS文件。这里的js文件作为中间层，提供前台控件和后台数据的绑定机制。 如图所示：  

![](./image/bind.png)   

![](./image/bind2.png)  

![](./image/bind3.png)  

创建出来的js文件如下所示：  

![](./image/bind4.png)   

### 2.6 工程运行调试

在调试运行工程之前，先运行DevTool\bin目录下的startpgsql.bat、startredis.bat、startzookeeper.bat启动PostgreSQL、Redis和Zookeeper

工程可以通过maven运行jetty的方式启动，右键工程选择运行方式->Maven build，在Goals文本框中输入“jetty:run”,点击运行，即可启动工程。 在浏览器中输入localhost:8080/XXX(XXX为artifactId)，可打开工程首页，界面如下：

![](./image/run.png)   

# 3 代码提示

## 3.1 标签助手

在“web页面编辑器”中，键入“<”即可开启智能提示助手，准确的提示出过滤后所有的标签，如图：  

![](./image/assistant.png)   

提示框左侧是经过过滤后的标签列表，右侧是当前标签对应的种类，适用浏览器版本，描述，备注以及示例，一目了然，方便快捷。 输入数字快捷键可以更加方便的把对应标签插入html当前位置。

## 3.2 代码块

在html中直接输入触发字符可提示代码块。 代码块是常用的代码组合，比如在js中输入$，回车，则可以自动输入document.getElementById(id)。 再举例，在HTML中输入i，回车，可以得到input button标签。  

![](./image/assistant2.png)   

如图，提示框左侧是过滤后的代码块列表，右侧是当前代码块的触发字符，来源，以及模板代码。

## 3.3 Emmet支持

web页面编辑器内嵌了emmet（即zencoding）插件。输入div#id1，按下tab，可以自动生成。Emmet的详细语法请查阅其官方网站emmet.io。  

## 3.4 JQuery支持

web页面编辑器内嵌了jquery插件，可以提示jquery语法。提示效果如下：  

![](./image/jquery.png)   

## 3.5 标签和代码块智能提示配置

标签可通过在窗口->首选项->iuap->Html标签扩展里面进行编辑，可创建自定义标签提示。  

![](./image/tagconfig.png)   

![](./image/snippet.png)   

高亮处为代码定义主体。 第一行单引号内为代码块显示标题，第二行trigger为代码块触发字符，第三行expansion为代码块主体。若想添加新的代码块，复制此处，粘贴在下行，并修改相应代码即可。

# 4 元数据设计器

## 4.1 元数据设计器概述

模型设计器作为实体模型、数据库模型的定义以及代码生成和脚本生成的工具，为业务系统中的领域模型的建立和基础代码的产生提供了一个完整的平台。业务分析和设计人员在建立领域模型时，其模型元素和模型设计器的实体操作等模型保持一致的描述方式，以保证领域模型顺利的建立和代码的顺利生成。

## 4.2 功能说明

系统提供实体模型、数据库模型等定义功能

提供版本控制和权限特性

提供模型的代码生成和数据库表的脚本生成功能

## 4.3 环境配置

从组件仓库下载元数据依赖的组件（maven依赖）。

![](./image/metadata23.png)

![](./image/metadata24.png)

安装完成后，确保maven项目pom.xml文件有元数据组件。

如下代码段：
```
	<dependency>
      <groupId>com.yonyou.iuap</groupId>
	  <artifactId>metadata-jdbc</artifactId>
	  <version>2.0.1-SNAPSHOT</version>
	</dependency> 
```

数据库连接配置

设置数据库连接配置信息

规则：一个项目配置一个数据库连接配置，具体位置：src/main/resources/application.properties

![](image/metadata28.png)

![](image/metadata29.png)
 
## 4.4 界面功能分布

根据界面功能分布，MDP模型设计器有5个重要的组成部分，分别是元数据管理器、画布、工具面板、属性面板和模型视图。

![](./image/metadata16.png)

### 4.4.1 元数据管理器

元数据管理器是提供元数据的相关管理工具，它支持元数据的创建、数据库导入实体组件、导入元数据bmf文件及删除元数据文件同时方便查看所有模型资源以及公共资源。
管理器的文件系统结构是按树形方式展开，反映组件的树型组织关系：

![](./image/metadata25.png)

#### 4.4.1.1 新建实体组件

创建元数据bmf文件。

![](./image/metadata26.png)

#### 4.4.1.2 数据库导入实体组件

支持从数据导入实体组件。

![](./image/metadata27.png)

#### 4.4.1.3 导入模型文件

支持导入元数据bmf文件。

#### 4.4.1.4 删除

删除元数据bmf文件

### 4.4.2 画布

模型设计器是一个图形化的工具，画布工具便于使用者进行类图建模的绘制。使用者从工具面板中拖动相应元素到画布类图中创建实体、枚举、值对象等。并可对类图中的对象进行直接增、删、改操作。

并且在画布类图中，选定类图中对象后点击鼠标右键，可以进行导入属性、发布元数据、发布元数据（忽略版本）、删除发布元数据、生成java源代码、导出pdm(.xml)文件等功能。上述操作的详细说明将在元数据典型应用章节中进行阐述。

### 4.4.3 工具面板

实体组件工具面板上用户通过拖拽的方式，在画布上创建实体组件、创建枚举、创建业务接口、创建注释以及创建关联关系。

### 4.4.4 属性面板

属性面板主要是在设计修改模型时，为使用者提供实时查性和设置当前所选定对象的基本属性的工具。

在面板的右上角有快捷应用功能按钮 ![](./image/metadata22.png)， 如下是对各按钮功能的描述：

![](./image/metadata21.png)[属性视图图钉功能]，选中后，固定当前选择对象，在使用者点击其它对象时，属性视图不随之改变；

![](./image/metadata18.png)[显示类别]，选中后，属性视图将按属性类型的方式排列展示；

![](./image/metadata19.png)[显示高级属性]，选中后，属性视图将显示高级属性；

![](./image/metadata20.png)[恢复缺省值]，选中后，恢复当前选择对象的属性的缺省值。

### 4.4.5 模型视图

在模型视图界面中可以对实体若干属性进行新增、修改和删除操作，还可以实现业务和属性映射。介绍实体属性时会具体说明。

## 4.5 创建组件

### 4.5.1 创建实体

#### 4.5.1.1 实体属性

实体可以有若干属性和操作，还可以实现接口。

![](./image/metadata1.png)

具体实体各属性

实体中属性的设置要点如下：

类型样式：目前支持Single、REF、LIST三种。

SINGLE 单一样式，最终的类型就是原始数据类型。

REF 引用样式，只用于实体，值对象，设置为引用样式，便可以得到引用类型。

LIST 列表样式类最终的类型为List<数据类型> 集合类型。

类型：可以自行选择，当建立实体间关系时，也会自动设置；主键一定要设置为ID类型。

目前元数据支持如下基本类型：String, Integer, BigInteger, MEMO, Double, Boolean, Date, Timestamp, Time, BigDecimal, ID, Money, BLOB, CLOB,CUSTOM,MULTILANGTEXT,IMAGE,FREEDOM,CONVERSIONRATE等。

字段名称：生成数据库表列的名称。

隐藏： 是否隐藏。

空：是否可以为空。

扩展标签：
1）cacheable标识此实体进行缓存处理。
2）自由配置供二次开发使用。

#### 4.5.1.2 实体的特性

实体的特性是从各种各样不同的实体中提取出来的一些共同的有特点的属性，比如说单据主实体上一定会有单据号，单据日期等。利用实体的特性，可以方便地构造出具有某些属性的实体，降低开发人员的工作量。

开发人员有两种方式来得到实体的特性：

1.可以通过修改iuap Studio\iuap\metadata\Features文件夹下的xml文件来增加新的特性，其中特性文件名为Features_模块名；

2.开发人员亦可通过导出特性的方法来导出特性。

![](./image/metadata4.png)

导出特性 点击后可以进入如下界面：

![](./image/metadata2.png)

特性界面

其中特性名称为该特性的显示名称。点击next后会进入选择要导出的特性的界面，选择要导出的特性，点击完成。

![](image/metadata3.png)

选择特性
  
#### 4.5.1.3 实体间关系

目前元数据的关系支持关联、组合。

关联，属性的类型样式是REF，在代码中生成的是一个String。

组合，属性的类型样式是LIST，在代码中生成的是一个list集合。

![](./image/metadata11.png)
实体间关系

设置关系的注意事项：

关系中必须设置源属性。

设置了关系后，属性的类型便会发生变化，但删除关系后，属性的类型不会恢复，务必要检查。另外，修改过带有关系的属性后，需要重新设置关系（即重新拉线）。

注意：对于组合关系，子实体中不需要指向主实体的外键属性，这是业务模型而不是数据库模型，在元数据发布时会自动在数据库中为子表加上外键。外键列的名称就是源属性（在上图中即为“订单”实体中的“订单明细”属性）对应的字段名称，所以建议将源属性对应的字段名称设为所期望的子表对应主表的外键名称。

如果实体是从PDM导入的，则务必按照注意事项修改模型。

![](./image/metadata12.png)
简单实体

简单VO简部全部为简单字段，与其它实体无任何关系，该实体除了客户类型为枚举类型，其它字段均为基本类型，和其它实体没有任何关系。

这是最简单的一种类型，依照实体默认产生数据库表,表名可以在设计器上指定，字段和实体属性保持一致即可，字段名也可以在设计器上设置。

![](./image/metadata13.png)

一对一关联

一个实体一对一关联了另一个实体，且在产生的JAVA类里关联源字段为字符串类型，而不是被关联实体的类型，我们把这种关联称之为关联，称关联源字段的数据类型为引用类型。

比如客户关联了地区，客户里有一个字段叫[客户所在地区]，称为关联源字段，如果是引用关联，则产生的客户JAVA类里，关联源字段为String类型，记录的是地区的主键ID。

![](./image/metadata14.png)

一对多组合

对于一对多组合，保存时除了保存主实体，还要保存被组合的子实体，删除时也一起删除，查询时提供全部查询和只查主实体对象。

### 4.5.2 创建业务接口

![](./image/metadata5.png)

业务接口

在设计器上，业务接口只能增加属性，相应会生成get方法，比如增加一个name属性，会生成一个getName()方法，方法的返回值可以自由选择。实际上，如果希望灵活运用业务接口，你可以摆脱设计器的限制，你在设计器上设计出一个业务接口，生成代码后，可以给业务接口增加任何方法，当然实现类也由你自己实现，此时只一通过元数据来管理你的业务接口而已。

业务接口的实现类可以分为三种情况，优先级从高到低排列如下：

1.实体和特定业务接口连线上的实现类

![](./image/metadata6.png)
实体实现接口连线

2.实体上注册的实现类

![](./image/metadata7.png)

实体注册实现类

3.业务接口上注册的实现类

![](./image/metadata8.png)

业务接口注册实现类

![](./image/metadata9.png)

根据映射关系自动生成

### 4.5.3 创建枚举

![](./image/metadata10.png)

创建枚举类型

创建一个枚举类型时，要注意以下几点：

枚举类型必须设置返回值类型，当前支持String和Integer两种

## 4.5 元数据典型应用

### 4.5.1 元数据发布

元数据发布的最小单元是组件。选中某个组件，或者在画布中点击右键，即可发布元数据。

元数据发布分为两种发布方式：1.发布元数据。2.发布元数据（忽略版本）。

![](./image/metadata15.png)

发布元数据

注意事项：

必须保存后才能发布

发布时会一起发布被依赖的组件，支持双向依赖，循环依赖。

如果发布元数据时忽略版本，则完全以本地的模型文件为准，可能造成库中新版本的模型被覆盖，非特殊情况不推荐使用。

### 4.5.2 删除发布的元数据

删除已发布在数据库中的元数据。

### 4.5.3 导出java代码

发布元数据后，在设计器上右键->导出java源代码可以导出java代码

### 4.5.4 导出sql脚本

发布元数据后，在设计器上右键->导出sql脚本

脚本带有实体建表语句及扩展表建表语句（ext_+原实体表名）。

### 4.5.5 导出PDM文件

如果当前文件夹下有同名的pdm，会分析PDM文件中的外键信息.并添加到新生成的PDM文件中。

# 5. 模版引擎

iuap Studio 提供模版引擎同时支持预置模版和用户自定义模版。模版引擎让用户不仅仅是使用者，还是创造者，赋予用户自主优化开发过程的能力。 

## 5.1 代码块/控件模版

studio初始安装后，打开html页面，左侧工具栏中预置了iuap布局与控件代码块。可直接拖拽进html编辑器中。 点击菜单栏中窗口->首选项->iuap->控件工具箱，可自定义控件的分类和内容。 自定义的控件代码存放在studio安装根目录下的templates/code中。

![](./image/code.png)

## 5.2 页面模版

点击工具栏新建->iuap页面模版,可以使用页面模版创建页面。 Studio初始安装后， 预置了通用和电商两个领域类别的模版，包含了近30套页面。用户可以根据场景选择模版，点击下一步，输入页面名称即可完成，生成的页面默认放在工程下的src/main/webapp/pages目录中。

![](./image/pagetemp.png)

用户也可以根据已有页面生成可用模版。在pages下已完成的页面目录点击右键，选择iuap Tools->生成页面模版，进入页面模版生成向导。 填写完成名称，分类等信息后，点击完成模版会生成在指定位置。这时，可以在新建->iuap页面模版中可以看到刚导出的页面模版了。

## 5.3 快速代码生成

代码片段通过代码块模版生成，前端页面通过页面模版生成，后端代码则可以通过快速代码生成实现。 右键点击工程，选择iuap Tools->快速代码生成， 进入代码生成向导。 首先选择数据源，这里有两种方式， 从元数据导入和从数据库导入。

![](./image/codegenerator.png)

选择元数据导入： 左侧树列出了工程所有元数据文件， 选择其中一个实体，右侧会出现实体的所有属性，选择要使用的属性，点击下一步，进入信息配置页面。 选择要使用的模版，然后在通用标签页中填写生成的类的包名和名称等，在自定义属性标签页输入代码中可能用到的键值对。点击完成即可。 

![](./image/codegenerator1.png)

![](./image/codegenerator2.png)

![](./image/codegenerator4.png)

选择数据库导入： 选择数据库导入之前应该确认是否为工程配置了正确的数据库连接。 右键工程选择iuap Tools->数据库连接配置， 进入数据源配置页面，填写数据库信息后，点击测试连接，如果通过，则可以选择数据库导入。 进入下一步，左侧是数据库中的table列表。选择其中一个table，右侧则列出table的所有列名。 勾选需要使用到的列名，点击下一步，进入信息配置页面，同元数据导入。 点击完成即可。 

![](./image/codegenerator3.png)

这里需要注意，Studio预置了两套代码模版，一个是iuap持久化模版，一个是JPA方式持久化模版。 用户也可以自定义模版。模版由两部分组成，描述文件和代码模版文件。

描述文件：description.xml，内容举例如下：

```
	<?xml version="1.0" encoding="UTF-8"?>
	<codeGenerator displayname="JPA" id="jpa" ext="false" xmlns="http://www.yonyou.com/studio/codeGenerator/description" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.yonyou.com/studio/codeGenerator/description description.xsd ">
	  <description>jpa</description>
	  <template>
	    <name>javaBean</name>
	    <displayname>实体类</displayname>
	    <resourcePath>jpa/main.java.vm</resourcePath>
	    <resourceType>class</resourceType>
	  </template>
	  <template>
	    <name>dao</name>
	    <displayname>DAO</displayname>
	    <resourcePath>jpa/dao.java.vm</resourcePath>
	    <resourceType>class</resourceType>
	  </template>
	</codeGenerator>
 ```

其中codeGenerator是根节点，属性有displayname(显示名)，id(唯一标识),ext(false代表预置模版，true为自定义模版)
description为模版的功能描述，介绍模版的功能和使用方式。
template节点为模版包含的模版内容文件，可以有多个，属性有name(英文名),displayname（中文显示名）,resourcePath(模版文件相对位置),resourceType(class为后台java类文件，resource为非java资源文件)。

代码模版文件：XXXX.vm，以生成实体类的模版文件main.java.vm举例：

```
	#set($tableClass=${customizer.getConfig("javaBean").className})
	#set($package=${customizer.getConfig("javaBean").codePackage})
	package ${package};
	
	import java.io.Serializable;
	import javax.persistence.*;
	${table.importStatements}
	
	/**
	 * The persistent class for the ${table.name} database table.
	 * 
	 */
	@Entity
	@Table(name="${table.name}")
	@NamedQuery(name="${tableClass}.findAll", query="SELECT ${table.aliasForQuery} FROM ${tableClass} ${table.aliasForQuery}")
	public class ${tableClass} implements Serializable {
		private static final long serialVersionUID = 1L;
	#####
	##### fields
	#####
	#foreach ($column in $table.fieldList)
	#if ($column.propertyName == $table.keyAttribute)
		@Id
	#end
		${column.fieldScope} ${column.simplePropertyType} ${column.propertyName};
	#end
	
		public ${tableClass}() {
		}
	
	#####
	##### simple properties getters and setters
	#####
	#foreach ($column in $table.fieldList)
	
		${column.propertyGetScope} ${column.simplePropertyType} ${column.propertyGetter}() {
			return this.${column.propertyName};
		}
	
		${column.propertySetScope} void ${column.propertySetter}(${column.simplePropertyType} ${column.propertyName}) {
			this.${column.propertyName} = ${column.propertyName};
		}
	
	#end
	}
```

其中`customizer.getConfig("javaBean")`表示名为javabean的模版文件。 javabean模版可见description.xml，代表实体类。模版文件有4个属性：className(类名称),codePackage(类所在包名),resourcePath(资源路径),resourceName(资源名称)。

由此，则`${customizer.getConfig("javaBean").className}`代表javabean模版文件生成的文件的类名称，这个是在信息配置页面用户配置的。

`table`代表选择的模型数据表。table有几个属性：name(表名),aliasForQuery(查询语句别名),keyAttribute(主键属性),fieldList(属性列表),importStatements(需要引用的类名),nameSpace(命名空间),metaDefinedName(元数据识别名).

由此，则`${table.name}`代表数据表名。`$table.fieldList`代表属性列表。 通过`#foreach ($column in $table.fieldList)`循环执行，循环体内column为具体字段属性。 column也有几个属性：fieldScope(属性作用域),simplePropertyType(属性对应java类型)，propertyName(属性名称)，propertyGetScope(属性get方法作用域)，propertyGetter(get方法名),propertySetScope(set方法作用域)，propertySetter(set方法名)。

关于自定义属性页中的键值对，举例说明：
添加一个key为company，value为yonyou的键值对，在模版文件中使用${properties.get("company")}即可获取value值yonyou，完成数据的替换。

模版文件的语法遵循velocity语法规则。

将包含描述文件和模版文件的文件夹放在Studio安装根目录下的templates/generator目录下，即可作为自定义代码生成模版识别。

# 6. 组件管理

iuap平台为组件化管理。 把基本功能按照组件拆分。通过maven仓库管理组件。Studio通过可视化界面， 提供maven组件的下载安装管理。

## 6.1 组件仓库

点击主菜单栏的工具->组件仓库，打开“组件商店”。 类似Eclipse MarketPlace的功能。 连接官网提供的组件仓库，按分类显示所有组件的详情描述，并提供下载安装按钮。Installed页签中列出本地maven库中已安装的所有iuap组件。

![](./image/component.png)

组件仓库页面提供Import(导入)按钮，可导入从官网下载的maven组件jar包安装到本地maven库中。

## 6.2 工程组件配置

右键点击工程，选择iuap Tools->工程组件。 显示本地所有已安装的iuap组件。 多选框中已勾选的表示工程使用的本地iuap组件，组件版本为下拉框文本，可选择使用版本并通过勾选iuap组件完成具体工程对iuap组件具体版本的使用。

![](./image/component1.png)