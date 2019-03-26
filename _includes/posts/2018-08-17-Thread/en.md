
> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！

# 多线程实现方式

## 异步多线程的三大特点

1.同步方法卡界面，原因是主线程被占用；异步方法不卡界面，原因是计算交给了别的线程，主线程空闲;

2.同步方法慢，原因是只有一个线程计算；异步方法快，原因是多个线程同时计算，但是更消耗资源，不宜太多;

3.异步多线程是无序的，启动顺序不确定、执行时间不确定、结束时间不确定


## Async  

```C#
private void DoSomethingLong(string name)
        {
            Console.WriteLine("****************DoSomethingLong Start {0}***************", Thread.CurrentThread.ManagedThreadId);
            long lResult = 0;
            for (int i = 0; i < 10000000; i++)
            {
                lResult += i;
            }
            Thread.Sleep(2000);

            Console.WriteLine("****************DoSomethingLong   End {0}***************", Thread.CurrentThread.ManagedThreadId);
    
        }
        

        {
         DoSomethingDelegate method = new DoSomethingDelegate(this.DoSomethingLong);
    
            IAsyncResult asyncResult = null;
    
            AsyncCallback callback = t =>
                {
                    Console.WriteLine(t.Equals(asyncResult));
    
                    Console.WriteLine(t.AsyncState);
                    Console.WriteLine("这里是回调函数 {0}", Thread.CurrentThread.ManagedThreadId);
                };
    
            //callback.Invoke(IAsyncResult)


            asyncResult = method.BeginInvoke("btnAsync_Click", callback, "假洒脱");//参数1 方法的参数 参数2 回调委托 参数3 回调状态 
            
            //asyncResult.IsCompleted 是否完成
            // method.EndInvoke(asyncResult); 回调
            }
```
## Thread
``` C#          
​            {

           List<Thread> threadList = new List<Thread>();
            for (int i = 0; i < 5; i++)
            {
                string name = string.Format("btnThread_Click_{0}", i);
                ThreadStart method = () => this.TestThread(name);
                Thread thread = new Thread(method);//1 默认前台线程：程序退出后，计算任务会继续
                thread.IsBackground = true;//2 后台线程：程序退出，计算立即结束
                thread.Start(); //线程进入就绪队列
                threadList.Add(thread);

                //ParameterizedThreadStart method = o => this.TestThread(o.ToString());
                //Thread thread = new Thread(method);
                //thread.Start(name);
            }

            foreach (Thread thread in threadList)
            {
                thread.Join();//当调用了 Thread.Join()方法后,当前线程会立即被执行
            }
    
}
​    
​                 /// <summary>
        /// 使用Thread 完成多线程回调
        /// </summary>
        /// <param name="method">要多线程执行的任务</param>
        /// <param name="callback">回调执行的任务</param>

        private void ThreadBeginInvoke(ThreadStart method, Action callback)
        {
            ThreadStart methodAll = new ThreadStart(() =>
            {
                method.Invoke();
                callback.Invoke();
            });
            Thread thread = new Thread(methodAll);
            thread.Start();
        }         
 ```       
        
## ThreadPool
 ``` C#     
        {
        static  void Main(string[] args)
        {
            ManualResetEvent mre = new ManualResetEvent(false);//创建ManualResetEvent信号量,主线这里构造函数必须传递false
            ThreadPool.QueueUserWorkItem((o => eowOne(mre)));//开启辅助线程
            mre.WaitOne();//让主线程等待子线程的完成
            Console.WriteLine("主线程继续做它的事情!");
            Console.Read();
        }

        /// <summary>
        /// 辅助线程一
        /// </summary>
        static void eowOne(ManualResetEvent mre)
        {
            var watch = Stopwatch.StartNew();
            Thread.Sleep(2000);
            watch.Stop();
            Console.WriteLine("辅助线程一做完了它的事,耗时:{0}", watch.ElapsedMilliseconds/1000);
            mre.Set();//告诉主线程子线程执行完了,如果不给ManualResetEvent实例调用这个方法,主线程会一直等待子线程调用ManualResetEvent实例的Set方法
        }
        }
```        
        
        
##  Task
``` C#     
        {
            
             TaskFactory taskFactory = new TaskFactory();

            for (int i = 0; i < 5; i++)
            {
                string name = string.Format("btnAsync_Click_{0}", i);
                Action act = () => this.TestThread(name);
                //taskFactory.StartNew(act);
                
                //Task task = new Task(act);
                //task.Start();

                Task task = Task.Run(act);
            }
        }
 ```
        
## Parallel 并行计算
 ``` c#       
        {
                        //Parallel.Invoke(() => this.TestThread("btnParallel_Click_0")
            //              , () => this.TestThread("btnParallel_Click_1")
            //              , () => this.TestThread("btnParallel_Click_2")
            //              , () => this.TestThread("btnParallel_Click_3")
            //              , () => this.TestThread("btnParallel_Click_4"));


            //等于使用4个task，然后主线程同步invoke一个委托  然后主线程waitall

            //Parallel.For(6, 10, t =>
            //{
            //    string name = string.Format("For btnParallel_Click_{0}", t);
            //    this.TestThread(name);
            //});

            //Parallel.ForEach(new int[] { 5, 6, 7, 10, 8473847 }, t =>
            //{
            //    string name = string.Format("ForEach btnParallel_Click_{0}", t);
            //    this.TestThread(name);
            //});

            ParallelOptions parallelOptions = new ParallelOptions()
            {
                MaxDegreeOfParallelism = 5
            };
            Parallel.For(6, 15, parallelOptions, (t, state) =>
                        {
                            string name = string.Format("btnParallel_Click_{0}", t);
                            this.TestThread(name);
                            //state.Break();//退出单次循环
                            //state.Stop();//退出全部的循环
                            //return;
                        });

            watch.Stop();
        }
 ```       
        
        