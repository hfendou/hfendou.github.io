

> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！



# WebApiThrottle限流框架使用手册
阅读目录：

[介绍](https://www.cnblogs.com/mushroom/p/4659200.html#one)

[基于IP全局限流](https://www.cnblogs.com/mushroom/p/4659200.html#two)

[基于IP的端点限流](https://www.cnblogs.com/mushroom/p/4659200.html#three)

[基于IP和客户端key的端点限流](https://www.cnblogs.com/mushroom/p/4659200.html#four)

[IP和客户端key的白名单](https://www.cnblogs.com/mushroom/p/4659200.html#five)

[IP和客户端key自定义限制频率](https://www.cnblogs.com/mushroom/p/4659200.html#six)

[端点自定义限制频率](https://www.cnblogs.com/mushroom/p/4659200.html#seven)

[关于被拒请求的计数器](https://www.cnblogs.com/mushroom/p/4659200.html#eight)

[在web.config或app.config中定义限制策略](https://www.cnblogs.com/mushroom/p/4659200.html#nine)

[获取API的客户端key](https://www.cnblogs.com/mushroom/p/4659200.html#ten)

[存储限流的数据](https://www.cnblogs.com/mushroom/p/4659200.html#eleven)

[运行期间更新限制频率](https://www.cnblogs.com/mushroom/p/4659200.html#twelve)

[限流的请求日志](https://www.cnblogs.com/mushroom/p/4659200.html#thirteen)

[用ThrottlingFilter、EnableThrottlingAttribute特性配置限制频率](https://www.cnblogs.com/mushroom/p/4659200.html#fourteen)

[关于ThrottlingMiddleware限制频率](https://www.cnblogs.com/mushroom/p/4659200.html#fifteen)