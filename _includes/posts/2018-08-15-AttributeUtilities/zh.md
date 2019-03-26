> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！

# 巧用反射+特性获取枚举描述信息


好比如枚举的用户状态，用0/1/2分别表示不同的用户状态。

```C#
public enum UserState
    {
        /// <summary>
        /// 正常
        /// </summary>
        [Remark("正常")]
        Normal = 0,
        /// <summary>
        /// 冻结
        /// </summary>
        [Remark("冻结")]
        Frozen = 1,
        /// <summary>
        /// 删除
        /// </summary>
        [Remark("删除")]
        Delete = 2
    }
```

封装特性，描述状态信息

 ```C#   
public class RemarkAttribute : Attribute
    {
        public RemarkAttribute(string remark)
        {
            _Remark = remark;
        }

        private string _Remark;

        public string Remark
        {
            get
            {
                return _Remark;
            }
        }        
    }
 ```

枚举的扩展方法，反射获取特性的描述

```C#      
 public static class RemarkExtend
    {
        public static string GetRemark(this Enum eValue)
        {
            Type type = eValue.GetType();
            FieldInfo field = type.GetField(eValue.ToString());
            RemarkAttribute remarkAttribute = (RemarkAttribute)field.GetCustomAttribute(typeof(RemarkAttribute));
            return remarkAttribute.Remark;
        }
    }
```     
 
调用时，如下：

 ```C#        
        ///  string remark = UserState.Normal.GetRemark();
```  