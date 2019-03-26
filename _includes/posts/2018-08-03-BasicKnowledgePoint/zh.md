

> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！


# C#/.NET 知识点累积在路上 #

## 泛型类的五大约束


1. where T：struct  限定当前参数类型必须是值类型
2. where T：class   限定当前类型参数类型必须是引用类型
3. where T：new（） 限定当前参数类型必须有一个无参构造器
4. where T：<base class name>   限定当前参数类型 必须是当前类  或者当前类为基类的类
5. where T：<interface name> 限定当前参数类型必须实现指定接口 


---




