

> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！



# WCF、WebAPI、WCFREST、WebService之间的区别


```markdown
在.net平台下，有大量的技术让你创建一个HTTP服务，像Web Service，WCF，现在又出了Web API。在.net平台下，你有很多的选择来构建一个HTTP Services。我分享一下我对Web Service、WCF以及Web API的看法。

　　Web Service

　　1、它是基于SOAP协议的，数据格式是XML

　　2、只支持HTTP协议

　　3、它不是开源的，但可以被任意一个了解XML的人使用

　　4、它只能部署在IIS上

 

　　WCF

　　1、这个也是基于SOAP的，数据格式是XML

　　2、这个是Web Service（ASMX）的进化版，可以支持各种各样的协议，像TCP，HTTP，HTTPS，Named Pipes, MSMQ.

　　3、WCF的主要问题是，它配置起来特别的繁琐

　　4、它不是开源的，但可以被任意一个了解XML的人使用

　　5、它可以部署应用程序中或者IIS上或者Windows服务中

 

　　WCF Rest

　　1、想使用WCF Rest service，你必须在WCF中使用webHttpBindings

　　2、它分别用[WebGet]和[WebInvoke]属性，实现了HTTP的GET和POST动词

　　3、要想使用其他的HTTP动词，你需要在IIS中做一些配置，使.svc文件可以接受这些动词的请求

　　4、使用WebGet通过参数传输数据，也需要配置。而且必须指定UriTemplate

　　5、它支持XML、JSON以及ATOM这些数据格式

 

　　Web API

　　1、这是一个简单的构建HTTP服务的新框架

　　2、在.net平台上Web API 是一个开源的、理想的、构建REST-ful 服务的技术

　　3、不像WCF REST Service.它可以使用HTTP的全部特点（比如URIs、request/response头，缓存，版本控制，多种内容格式）

　　4、它也支持MVC的特征，像路由、控制器、action、filter、模型绑定、控制反转（IOC）或依赖注入（DI），单元测试。这些可以使程序更简单、更健壮

　　5、它可以部署在应用程序和IIS上

　　6、这是一个轻量级的框架，并且对限制带宽的设备，比如智能手机等支持的很好

　　7、Response可以被Web API的MediaTypeFormatter转换成Json、XML 或者任何你想转换的格式。

　　

　　WCF和WEB API我该选择哪个？

　　1、当你想创建一个支持消息、消息队列、双工通信的服务时，你应该选择WCF

　　2、当你想创建一个服务，可以用更快速的传输通道时，像TCP、Named Pipes或者甚至是UDP（在WCF4.5中）,在其他传输通道不可用的时候也可以支持HTTP。

　　3、当你想创建一个基于HTTP的面向资源的服务并且可以使用HTTP的全部特征时（比如URIs、request/response头，缓存，版本控制，多种内容格式），你应该选择Web API

　　4、当你想让你的服务用于浏览器、手机、iPhone和平板电脑时，你应该选择Web API

　　

　　原文：http://www.dotnet-tricks.com/Tutorial/webapi/JI2X050413-Difference-between-WCF-and-Web-API-and-WCF-REST-and-Web-Service.html

所有问题都会有一定程度的抽象和假设

```