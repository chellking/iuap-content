# 消息推送组件概述 #

## 业务需求 ##

消息推送主要用户提升用户体验，避免用户刷新页面从服务器端拉取数据。例如邮件提醒，重要通知能即时推送给用户，提高用户体验。

## 功能说明 ##

iuap-message组件提供了向手机发送短信、发送电子邮件、向APP推送消息的功能。

1.	提供了向手机发送短信
2.	提供了发送电子邮件
3.	电子邮件支持抄送、密送
4.	电子邮件支持附件发送
3.	提供了向APP推送消息的功能
4.	可扩展的消息发送方式；

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，其对外提供的依赖方式如下：
```
	<dependency>
      <groupId>com.yonyou.iuap</groupId>
      <artifactId>iuap-message</artifactId>
      <version>${iuap.modules.version}</version>
    </dependency>
```
${iuap.modules.version} 为平台在maven私服上发布的组件的version。

## 功能说明 ##

iuap-message组件提供了向手机发送短信、发送电子邮件、向APP推送消息的功能。

1. 消息推送  
在公司现有的消息推送平台的基础上，利用此平台的REST服务，将业务数据包装成符合要求的格式，发送到平台上，完成APP的推送。（用户的移动端也需要安装平台提供的SDK）  
2. 发送短信  
发送短信的功能是基于公司的短信推送平台，通过此平台的提供的REST服务，按照平台的要求，将数据封装提交到平台，完成短信发送任务。目前支持国内的移动、联通和电信三大网络。  
3. 发送邮件  
电子邮件服务，是基于JavaMail实现的邮件发送服务，使用JDK原生的javax.mail组件来完成发送邮件的服务。

主要工作流程如下： 	
1. 验证登录权限Authenticator   
2. 根据设置的Properties和Authenticator创建一个Session  
3. 创建一个MimeMessage实例，设置这个message的收信人、主题、内容等 
4. 发送邮件Transport.send(message);  
 
# 使用说明 #

## 组件包说明 ##

iuap-message组件提供了向手机发送短信、发送电子邮件、向APP推送消息的功能。

##组件配置##

将配置文件message-senderInfo.xml(可在上文示例工程拿到)放到工程的classpath下，如果是maven工程，放在src/main/resources目录下即可

## 工程样例 ##

消息推送组件提供有示例工程iuap-message-example，用户可从maven库上下载，示例工程中有较为完整的对iuap-message组件的使用示例代码。

## 开发步骤 ##

*可直接参考以下示例工程：*


1 . 添加配置文件：  

将配置文件message-senderInfo.xml(可在上文示例工程拿到)放到工程的classpath下，如果是maven工程，放在src/main/resources目录下即可  

2 . 调用消息服务的接口方法，发送消息:

```
    // 创建消息接收者
    MessageReceiver emailReceivers = new EmailReceiver("username1@domain.com,username2@domain.com");
    
    // 创建消息内容
    MessageContent emailContent = new EmailContent("我是标题1", "测试内容1");
    
    // 发送消息
    List<MessageResponse> responseList = new MessageSend(emailReceivers, emailContent).send(); 
```

## 常用接口 ##

**描述**  
短信、邮件和APP消息推送接口  

**请求方法**  
new MessageSend(msgReceivers, msgContent).send();  

**请求参数说明**  

<table>
  <tr>
    <th><br>  参数字段<br>  </th>
    <th><br>  必选<br>  </th>
    <th><br>  类型<br>  </th>
    <th><br>  长度<br>  </th>
    <th><br>  说明<br>  </th>
  </tr>
  <tr>
    <td><br>  msgReceivers<br>  </td>
    <td><br>  True<br>  </td>
    <td><br>  MessageReceiver<br>  </td>
    <td><br>  -<br>  </td>
    <td><br>  消息的接收者，有多个的时候，按照英文”,”分割<br>  </td>
  </tr>
  <tr>
    <td><br>  msgContent<br>  </td>
    <td><br>  True<br>  </td>
    <td><br>  MessageContent<br>  </td>
    <td><br>  -<br>  </td>
    <td><br>  消息内容，包含消息标题，消息内容，发送时间。其中消息标题和发送时间非必填<br>  </td>
  </tr>
</table>
- msgReceivers消息接收者。可以是邮件地址，或者是电话号码，或者是APP ID。
- msgContent消息内容，可以设置消息标题、文本内容。

## 扩展机制 ##
发送HTML内容的电子邮件：  
在设置邮件发送内容时，可以直接编写HTML代码，如下：  
 ```     
    StringBuffer htmlContent = new StringBuffer();
    htmlContent.append("<h1>我是标题</h1>");
    htmlContent.append("<h3>企业互联网运营支撑平台</h3>");
    htmlContent.append("<div><img src='http://img4.3lian.com/sucai/img6/230/29.jpg'></div>");
    htmlContent.append("<a href='http://ieop.yyuap.com/'>用友应用支撑平台</a>");
    MessageContent mailContent = new EmailContent("HTML Mail测试", htmlContent.toString());
```
这样就可以发送HTML格式的邮件。  
