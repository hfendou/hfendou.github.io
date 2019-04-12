

> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！



# 23DesignPattern(持续更新)

# 创建型

## 工厂模式（AbstractFactory）
``` c#
   public abstract class CarFactory
   {
        public abstract Car CarCreate();
    }
   public abstract class Car
     {
        public abstract void StartUp();
        public abstract void Run();
        public abstract void Stop();

    }
   public class HongQiCarFactory : CarFactory
     {
        public override Car CarCreate()
        {
            return new HongQiCar();
        }
    }
   public class BMWCarFactory : CarFactory
     {
        public override Car CarCreate()
        {
            return new BMWCar();
        }
    }
   public class HongQiCar : Car
     {
       public override void StartUp()
       {
           Console.WriteLine("Test HongQiCar start-up speed!");
       }
       public override void Run()
       {
           Console.WriteLine("The HongQiCar run is very quickly!");
       }
       public override void Stop()
       {
           Console.WriteLine("The slow stop time is 3 second ");
       }
    }
   public class BMWCar : Car
     {
        public override void StartUp()
        {
            Console.WriteLine("The BMWCar start-up speed is very quickly");
        }
        public override void Run()
        {
            Console.WriteLine("The BMWCar run is quitely fast and safe!!!");
        }
        public override void Stop()
        {
            Console.WriteLine("The slow stop time is 2 second");
        }
     }

```
```C#
            Console.WriteLine("Please Enter Factory Method No:");
            Console.WriteLine("******************************");
            Console.WriteLine("no         Factory Method");
            Console.WriteLine("1          HongQiCarFactory");
            Console.WriteLine("2          BMWCarFactory");
             Console.WriteLine("******************************");
             int no = Int32.Parse(Console.ReadLine().ToString());
             string factoryType = ConfigurationManager.AppSettings["No" + no];
             //CarFactory factory = new HongQiCarFactory();
             CarFactory factory = (CarFactory)Assembly.Load("FactoryMehtod").CreateInstance("FactoryMehtod." + factoryType); ;
             Car car = factory.CarCreate();
             car.StartUp();
             car.Run();
             car.Stop();
```


## 抽象工厂模式（AbstractFactory）
```c# 
 /// <summary>
    /// （AbstractFactory）抽象工厂类：提供创建不同品牌的手机屏幕和手机主板
    /// </summary>
    public abstract class AbstractFactory
    {
        //工厂生产屏幕
        public abstract Screens CreateScreen();
        //工厂生产主板
        public abstract MotherBoard CreateMotherBoard();
    }
    /// <summary>
    /// (AbstractProduct)屏幕抽象类：提供每一品牌的屏幕的继承
    /// </summary>
    public abstract class Screens
    {
        public abstract void print();
    }
    /// <summary>
    ///(AbstractProduct) 主板抽象类：提供每一品牌的主板的继承
    /// </summary>
    public abstract class MotherBoard
    {
        public abstract void print();
    }
    /// <summary>
    ///(Prouduct) 苹果手机屏幕
    /// </summary>
    public class AppleScreen : Screens
    {
        public override void print()
        {
            Console.WriteLine("苹果手机屏幕！");
        }
    }
    /// <summary>
    ///(Prouduct) 苹果手机主板
    /// </summary>
    public class AppleMotherBoard : MotherBoard
    {
        public override void print()
        {
            Console.WriteLine("苹果手机主板！");
        }
    }
    /// <summary>
    /// (Prouduct)小米手机屏幕
    /// </summary>
    public class XiaoMiScreen : Screens
    {
        public override void print()
        {
            Console.WriteLine("小米手机屏幕！");
        }
    }
    /// <summary>
    ///(Prouduct) 小米手机主板类
    /// </summary>
    public class XiaoMiMotherBoard : MotherBoard
    {
        public override void print()
        {
            Console.WriteLine("小米手机主板！");
        }
    }
    /// <summary>
    /// (ConcreateFactory)苹果手机工厂
    /// </summary>
    public class AppleFactory : AbstractFactory
    {
        /// <summary>
        /// 生产苹果手机屏幕
        /// </summary>
        /// <returns></returns>
        public override Screens CreateScreen()
        {
            return new AppleScreen();
        }

        /// <summary>
        /// 生产苹果手机主板
        /// </summary>
        /// <returns></returns>
        public override MotherBoard CreateMotherBoard()
        {
            return new AppleMotherBoard();
        }
    }
    /// <summary>
    /// (ConcreateFactory)小米手机工厂类
    /// </summary>
    public class XiaoMiFactory : AbstractFactory
    {
        /// <summary>
        /// 生产小米手机屏幕
        /// </summary>
        /// <returns></returns>
        public override Screens CreateScreen()
        {
            return new XiaoMiScreen();
        }

        /// <summary>
        /// 生产小米手机主板
        /// </summary>
        /// <returns></returns>
        public override MotherBoard CreateMotherBoard()
        {
            return new XiaoMiMotherBoard();
        }
    }

```

```c#

            //小米工厂生产小米手机的屏幕和主板
            AbstractFactory xiaomiFactory = new XiaoMiFactory();
            Screens xiaomiScreen = xiaomiFactory.CreateScreen();
            xiaomiScreen.print();
            MotherBoard xiaomiMotherBoard = xiaomiFactory.CreateMotherBoard();
            xiaomiMotherBoard.print();

            //苹果工厂生产苹果手机屏幕和主板
            AbstractFactory appleFactory = new AppleFactory();
            Screens appleScreen = appleFactory.CreateScreen();
            appleScreen.print();
            MotherBoard appleMotherBoard = appleFactory.CreateMotherBoard();
            appleMotherBoard.print();

            Console.Read();
```

## 建造者模式

```c#
/// <summary>
    /// Builder 
    /// </summary>
    public class Boss
    {
        public void MakeEmployee(Employee employee)
        {
            employee.BuildFrame();
            employee.BuildLenses();
        }
    }
    //创建员工 (Director)
    public abstract class Employee
    {
        //生产镜片
        public abstract void BuildLenses();
        //生产镜框
        public abstract void BuildFrame();
        //组装眼镜
        public abstract Glasses GetGlasses();
    }
    /// <summary>
    /// prouduct
    /// </summary>
    public class Glasses
    {
        private IList<string> parts = new List<string>();
        public void Add(string part)
        {
            parts.Add(part);
        }
        public void Show()
        {
            Console.WriteLine("开始组装眼镜了！");
            foreach (string part in parts)
            {
                Console.WriteLine("组件" + part + "装好了");
            }
            Console.WriteLine("组装完成！");
        }
    }
    ///ConcreateBuilder
    public class Employee1 : Employee
    {
        Glasses glasses = new Glasses();
        public override void BuildLenses()
        {
            glasses.Add("暴龙镜片");
        }
        public override void BuildFrame()
        {
            glasses.Add("暴龙镜框");
        }
        public override Glasses GetGlasses()
        {
            return glasses;
        }
    }
    /// <summary>
    /// ConcreateBuilder
    /// </summary>
    public class Employee2 : Employee
    {
        Glasses glasses = new Glasses();
        public override void BuildLenses()
        {
            glasses.Add("宝岛镜片");
        }
        public override void BuildFrame()
        {
            glasses.Add("宝岛镜框");
        }
        public override Glasses GetGlasses()
        {
            return glasses;
        }
    }
```
```C#
            Boss boss = new Boss();
            Employee employee1 = new Employee1();
            Employee employee2 = new Employee2();
            boss.MakeEmployee(employee1);
            Glasses glasses = employee1.GetGlasses();
            glasses.Show();
            boss.MakeEmployee(employee2);
            glasses = employee2.GetGlasses();
            glasses.Show();

            Console.ReadKey();
```

## 原型模式
``` c# 
 /// <summary>
    /// The 'Prototype' abstract class
    /// </summary>
    public abstract class Prototype
    {
        private string _id;

        /// <summary>
        /// Constructor
        /// </summary>
        public Prototype(string id)
        {
            this._id = id;
        }

        /// <summary>
        /// Gets id
        /// </summary> 
        public string Id
        {
            get { return _id; }
        }

        public abstract Prototype Clone();
    }

    public class ConcretePrototype1 : Prototype
    {
        /// <summary>
        /// Constructor
        /// </summary>
        public ConcretePrototype1(string id)
            : base(id)
        {
        }

        /// <summary>
        /// Returns a shallow copy
        /// </summary>
        /// <returns></returns>
        public override Prototype Clone()
        {
            return (Prototype)this.MemberwiseClone();
        }
    }

    public class ConcretePrototype2 : Prototype
    {
        /// <summary>
        /// Constructor
        /// </summary>
        public ConcretePrototype2(string id)
            : base(id)
        {
        }

        /// <summary>
        /// Returns a shallow copy
        /// </summary>
        /// <returns></returns>
        public override Prototype Clone()
        {
            return (Prototype)this.MemberwiseClone();
        }
    }
```

```C#
     ConcretePrototype1 p1 = new ConcretePrototype1("I");
     ConcretePrototype1 c1 = (ConcretePrototype1)p1.Clone();
     Console.WriteLine("Cloned: {0}", c1.Id);

     ConcretePrototype2 p2 = new ConcretePrototype2("II");
     ConcretePrototype2 c2 = (ConcretePrototype2)p2.Clone();
     Console.WriteLine("Cloned: {0}", c2.Id);
```


# 单例
## 单线程Singleton实现
```c# 
 class SingleThread_Singleton
    {
        private static SingleThread_Singleton instance = null;
        private  SingleThread_Singleton(){}
        public static SingleThread_Singleton Instance
        {
            get
            {
                if (instance == null)
                {
                    instance = new SingleThread_Singleton();
                }
                return instance;
            }
        }
    }
```

## 多线程Singleton实现
```c#
        private static volatile MultiThread_Singleton instance = null;
        private static object lockHelper = new object();
        private MultiThread_Singleton() { }
        public static MultiThread_Singleton Instance
        {
            get
            {
                if (instance == null)
                {
                    lock (lockHelper)
                    {
                        if (instance == null)
                        {
                            instance = new MultiThread_Singleton();
                        }
                    }
                }
                return instance;
            }
        }
```

## 静态Singleton实现
```c#
        public static readonly Static_Singleton instance = new Static_Singleton();
        private Static_Singleton()
        {
            TestClass.TestMethod<Static_Singleton>(this);
        }

//以上代码展开等同于

public static readonly Static_Singleton instance;
static Static_Singleton()
{
    instance = new Static_Singleton();
}
private Static_Singleton() { }


```

```c#
           TaskFactory taskFactory = new TaskFactory();
            List<Task> taskList = new List<Task>();
            for (int i = 0; i < 10000; i++)
            {
                taskList.Add(taskFactory.StartNew(() =>
                {
                    SingleThread_Singleton singleton = SingleThread_Singleton.Instance; //new Singleton();                   
                }));
            }
            Task.WaitAll(taskList.ToArray());

            TaskFactory tFactory = new TaskFactory();
            List<Task> tlist = new List<Task>();
            for (int i = 0; i < 10000; i++)
            {
                tlist.Add(tFactory.StartNew(() => {
                    Static_Singleton single= Static_Singleton.instance;
                }
                    ));
            }
            Task.WaitAll(tlist.ToArray());
```

# 结构型

## 装饰者模式
```C#
/// <summary>
    /// 手机抽象类，即装饰者模式中的抽象组件类
    /// </summary>
    public abstract class Phone
    {
        public abstract void Print();
    }

    /// <summary>
    /// 苹果手机，即装饰着模式中的具体组件类
    /// </summary>
    public class ApplePhone : Phone
    {
        /// <summary>
        /// 重写基类方法
        /// </summary>
        public override void Print()
        {
            Console.WriteLine("开始执行具体的对象——苹果手机");
        }
    }

    /// <summary>
    /// 装饰抽象类,要让装饰完全取代抽象组件，所以必须继承自Photo
    /// </summary>
    public abstract class Decorator : Phone
    {
        private Phone phone;

        public Decorator(Phone p)
        {
            this.phone = p;
        }

        public override void Print()
        {
            if (phone != null)
            {
                phone.Print();
            }
        }
    }

    /// <summary>
    /// 贴膜，即具体装饰者
    /// </summary>
    public class Sticker : Decorator
    {
        public Sticker(Phone p)
            : base(p)
        {
        }

        public override void Print()
        {
            base.Print();

            // 添加新的行为
            AddSticker();
        }

        /// <summary>
        /// 新的行为方法
        /// </summary>
        public void AddSticker()
        {
            Console.WriteLine("现在苹果手机有贴膜了");
        }
    }

    /// <summary>
    /// 手机挂件
    /// </summary>
    public class Accessories : Decorator
    {
        public Accessories(Phone p)
            : base(p)
        {
        }

        public override void Print()
        {
            base.Print();

            // 添加新的行为
            AddAccessories();
        }

        /// <summary>
        /// 新的行为方法
        /// </summary>
        public void AddAccessories()
        {
            Console.WriteLine("现在苹果手机有漂亮的挂件了");
        }
    }
```

```c#
 // 我买了个苹果手机
            Phone phone = new ApplePhone();

            // 现在想贴膜了
            Decorator applePhoneWithSticker = new Sticker(phone);
            // 扩展贴膜行为
            applePhoneWithSticker.Print();
            Console.WriteLine("----------------------\n");

            // 现在我想有挂件了
            Decorator applePhoneWithAccessories = new Accessories(phone);
            // 扩展手机挂件行为
            applePhoneWithAccessories.Print();
            Console.WriteLine("----------------------\n");

            // 现在我同时有贴膜和手机挂件了
            Sticker sticker = new Sticker(phone);
            Accessories applePhoneWithAccessoriesAndSticker = new Accessories(sticker);
            applePhoneWithAccessoriesAndSticker.Print();
            Console.ReadLine();
```

# 行为型

待更新