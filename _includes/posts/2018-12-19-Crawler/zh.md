

> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！


# c#利用WebClient和WebRequest获取网页源代码的比较
## WebClient类获取网页源代码
### WebClient类
　
　WebClient类位于System.Net命名空间下，WebClient类提供向URI标识的任何本地、Intranet或Internet资源发送数据以及从这些资源接收数据的公共方法。
源代码
```c#
///引用命名空间
using System.IO;
using System.Net;
using System.Text;


PageUrl = "http://www.webkaka.com"; //需要获取源代码的网页
WebClient wc = new WebClient(); // 创建WebClient实例提供向URI 标识的资源发送数据和从URI 标识的资源接收数据
wc.Credentials = CredentialCache.DefaultCredentials; // 获取或设置用于对向 Internet 资源的请求进行身份验证的网络凭据。

///方法一：
Encoding enc = Encoding.GetEncoding("GB2312"); // 如果是乱码就改成 utf-8 / GB2312
Byte[] pageData = wc.DownloadData(PageUrl); // 从资源下载数据并返回字节数组。
ContentHtml.Text = enc.GetString(pageData); // 输出字符串(HTML代码)，ContentHtml为Multiline模式的TextBox控件
   
/// 方法二：
/// ***************代码开始**********
/// Stream resStream = wc.OpenRead(PageUrl); //以流的形式打开URL
/// Encoding enc = Encoding.GetEncoding("GB2312"); // 如果是乱码就改成 utf-8 / GB2312
/// StreamReader sr = new StreamReader(resStream,enc); //以指定的编码方式读取数据流
/// ContentHtml.Text = sr.ReadToEnd(); //输出(HTML代码)，ContentHtml为Multiline模式的TextBox控件
/// resStream.Close();
/// **************代码结束********
/// 
wc.Dispose();
```

## WebRequest类获取网页源代码

## WebRequest类

WebRequest类是.NET Framework中“请求/响应”模型的abstract基类，用于访问Internet数据。使用WebRequest类请求/响应模型的应用程序可以用协议不可知的方式从Internet请求数据，在这种方式下，应用程序处理WebRequest类的实例，而协议特定的子类则执行请求的具体细节，请求从应用程序发送到某个特定的URI，如服务器上的网页。URI从一个为应用程序注册的WebRequest子类列表中确定要创建的适当子类。注册WebRequest子类通常是为了处理某个特定的协议（如HTTP或FTP），但是也可以注册它以处理对特定服务器或服务器上的路径的请求。
WebRequest类中最常用的是Create方法，Create方法用于为指定的URI方案初始化新的WebRequest实例。
　　
语法：
```c#
public static WebRequest Create
(
　 string requestUriString
)
　　参数：
　　requestUriString：标识Internet资源的URI。
　　返回值：特定URI方案的WebRequest子类。
　　注意：Create方法将运行时确定的WebRequest类的子类作为与requestUri最接近的注册匹配项返回。例如，当以http://开头的URI在requestUri中传递时，由Create返回一个HttpWebRequest。如果改为传递以file://开头的URI，则Create方法将返回FileWebRequest实例。.NET Framework包括对http://和file:// URI方案的支持。
```

## WebResponse类
```
　　WebResponse类是abstract基类，协议特定的响应类从该抽象基类派生。应用程序可以使用 WebResponse类的实例以协议不可知的方式参与请求和响应事务，而从WebResponse类派生的协议特定的类携带请求的详细信息。
　　在WebResponse类中最常用的是GetResponse方法，GetResponse方法用于当在子类中被重写时，返回对 Internet 请求的响应
　　语法：
　　public virtual WebResponse GetResponse ()
　　返回值：包含对Internet请求的响应的WebResponse。
```
源代码
```c#
///引用命名空间
using System.IO;
using System.Net;
using System.Text;
PageUrl = "http://www.webkaka.com"; //需要获取源代码的网页
WebRequest request = WebRequest.Create(PageUrl); //WebRequest.Create方法，返回WebRequest的子类HttpWebRequest
WebResponse response = request.GetResponse(); //WebRequest.GetResponse方法，返回对 Internet 请求的响应
Stream resStream = response.GetResponseStream(); //WebResponse.GetResponseStream 方法，从 Internet 资源返回数据流。 
Encoding enc = Encoding.GetEncoding("GB2312"); // 如果是乱码就改成 utf-8 / GB2312
StreamReader sr = new StreamReader(resStream, enc); //命名空间:System.IO。 StreamReader 类实现一个 TextReader (TextReader类，表示可读取连续字符系列的读取器)，使其以一种特定的编码从字节流中读取字符。 
ContentHtml.Text = sr.ReadToEnd(); //输出(HTML代码)，ContentHtml为Multiline模式的TextBox控件
resStream.Close(); 
sr.Close();

```