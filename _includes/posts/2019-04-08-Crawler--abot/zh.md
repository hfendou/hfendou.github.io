

> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！



# .Net开源网络爬虫Abot介绍

原文  http://www.cnblogs.com/JustRun1983/p/abot-crawler.html

主题 .Net jQuery

.Net中也有很多很多开源的爬虫工具，abot就是其中之一。Abot是一个开源的.net爬虫，速度快，易于使用和扩展。项目的地址是 https://code.google.com/p/abot/
对于爬取的Html，使用的分析工具是CsQuery, CsQuery可以算是.net中实现的Jquery, 可以使用类似Jquery中的方法来处理html页面。CsQuery的项目地址是https://github.com/afeiship/CsQuery

## 一. 对Abot爬虫配置
### 1. 通过属性设置

先创建config对象，然后设置config中的各项属性:
``` c#
CrawlConfiguration crawlConfig = new CrawlConfiguration(); 
crawlConfig.CrawlTimeoutSeconds = 100; 
crawlConfig.MaxConcurrentThreads = 10; 
crawlConfig.MaxPagesToCrawl = 1000; 
crawlConfig.UserAgentString = "abot v1.0 http://code.google.com/p/abot"; 
crawlConfig.ConfigurationExtensions.Add("SomeCustomConfigValue1", "1111"); 
crawlConfig.ConfigurationExtensions.Add("SomeCustomConfigValue2", "2222");
```
### 2. 通过App.config配置
直接从配置文件中读取，但是也任然可以在修改各项属性:

``` c#
CrawlConfiguration crawlConfig = AbotConfigurationSectionHandler.LoadFromXml().Convert(); 
crawlConfig.CrawlTimeoutSeconds = 100; 
crawlConfig.MaxConcurrentThreads = 10;
```

### 3. 应用配置到爬虫对象

```c#
PoliteWebCrawler crawler = new PoliteWebCrawler();
PoliteWebCrawler crawler = new PoliteWebCrawler(crawlConfig, null, null, null, null, null, null, null);
```

## 二，使用爬虫，注册各种事件

爬虫中主要是4个事件, 页面爬取开始、页面爬取失败、页面不允许爬取事件、页面中的链接不允许爬取事件。
下面是示例代码:

``` c#
crawlergeCrawlStartingAsync += crawler_ProcessPageCrawlStarting;//单个页面爬取开始 
crawler.PageCrawlCompletedAsync += crawler_ProcessPageCrawlCompleted;//单个页面爬取结束 
crawler.PageCrawlDisallowedAsync += crawler_PageCrawlDisallowed;//页面不允许爬取事件 
crawler.PageLinksCrawlDisallowedAsync += crawler_PageLinksCrawlDisallowed;//页面链接不允许爬取事件

void crawler_ProcessPageCrawlStarting(object sender, PageCrawlStartingArgs e)
{  PageToCrawl pageToCrawl = e.PageToCrawl;  Console.WriteLine("About to crawl link {0} which was found on page {1}", pageToCrawl.Uri.AbsoluteUri, pageToCrawl.ParentUri.AbsoluteUri);
}
void crawler_ProcessPageCrawlCompleted(object sender, PageCrawlCompletedArgs e)
{  CrawledPage crawledPage = e.CrawledPage;  if (crawledPage.WebException != null || crawledPage.HttpWebResponse.StatusCode != HttpStatusCode.OK)    Console.WriteLine("Crawl of page failed {0}", crawledPage.Uri.AbsoluteUri);  else    Console.WriteLine("Crawl of page succeeded {0}", crawledPage.Uri.AbsoluteUri);  if (string.IsNullOrEmpty(crawledPage.Content.Text))    Console.WriteLine("Page had no content {0}", crawledPage.Uri.AbsoluteUri);

}
void crawler_PageLinksCrawlDisallowed(object sender, PageLinksCrawlDisallowedArgs e)
{  CrawledPage crawledPage = e.CrawledPage;  Console.WriteLine("Did not crawl the links on page {0} due to {1}", crawledPage.Uri.AbsoluteUri, e.DisallowedReason);
}
void crawler_PageCrawlDisallowed(object sender, PageCrawlDisallowedArgs e)
{  PageToCrawl pageToCrawl = e.PageToCrawl;  Console.WriteLine("Did not crawl page {0} due to {1}", pageToCrawl.Uri.AbsoluteUri, e.DisallowedReason);
}
```
## 三， 为爬虫添加多个附加对象

Abot应该是借鉴了Asp.net MVC中的ViewBag, 也为爬虫对象设置了对象级别的CrwalBag和Page级别的ViewBag.

```c#
PoliteWebCrawler crawler = new PoliteWebCrawler();
crawler.CrawlBag.MyFoo1 = new Foo();//对象级别的CrwalBag
crawler.CrawlBag.MyFoo2 = new Foo();
crawler.PageCrawlStartingAsync += crawler_ProcessPageCrawlStarting;
...void crawler_ProcessPageCrawlStarting(object sender, PageCrawlStartingArgs e)
{  //获取CrwalBag中的对象  CrawlContext context = e.CrawlContext;  context.CrawlBag.MyFoo1.Bar();//使用CrwalBag  context.CrawlBag.MyFoo2.Bar();
  //使用页面级别的PageBag  e.PageToCrawl.PageBag.Bar = new Bar();
}
```

## 四，启动爬虫
启动爬虫非常简单，调用Crawl方法，指定好开始页面，就可以了。

```c#
CrawlResult result = crawler.Crawl(new Uri("http://localhost:1111/"));
if (result.ErrorOccurred)
        Console.WriteLine("Crawl of {0} completed with error: {1}", result.RootUri.AbsoluteUri, result.ErrorException.Message);else
        Console.WriteLine("Crawl of {0} completed without error.", result.RootUri.AbsoluteUri);
```

## 五，介绍CsQuery

在PageCrawlCompletedAsync事件中, e.CrawledPage.CsQueryDocument就是一个CsQuery对象。
这里介绍一下CsQuery在分析Html上的优势:
cqDocument.Select(".bigtitle > h1")
这里的选择器的用法和Jquery完全相同，这里是取class为.bittitle下的h1标签。如果你能熟练的使用Jquery，那么上手CsQuery会非常快和容易。
