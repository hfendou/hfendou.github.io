

> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！



## WebApi Route


```markdown
//路由配置 默认是 控制器/参数
 config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }

方法名 头用Get/Post/Put/Delete命名 可以忽略特性 ，但建议不要偷懒




HttpPost  action不能重复！！

public string PostStrs( string[] str) 
参数方式json 比如["dd","00"] 

public string PostStr(string str) 
单独参数 后台调用需要参数拼接url 暂时发现未能在表单body传参 

public string PostStr([FromBody]string str) 
Dictionary<string, string> dic = new Dictionary<string, string>();
 dic.Add("", "dfdsfs"); //左手边的key 必需为空
public string PostObjectToStr(Parm parms) json 传参 对象名要与json不一致 
dic分别添加对象的属性 对象名不一定要一致 属性名一定一致
不能dic 直接传对象

 public string PostJObject(JObject jData) 跟封装对象一样
 
 相同的方法名   throw 抛错  
 postman 抛错 "Message": "An error has occurred.",

public string PostDynamic(dynamic jData) 跟封装对象一样
public string PostStrAndInt(string str,int iValue) 只能url 拼接参数
public string PostStrList(List<int> iList,string strValue) 引用类型 json传参 值类型url传值

被标记为[FromBody]的参数只允许出现一次， 被标记为[FromUri]的参数可以出现多次

```

ps：使用swagger  会接触到三种参数类型
 query  url ?key=values
path   url {values}
body   值类型-->'' ；多个参数封装对象

---