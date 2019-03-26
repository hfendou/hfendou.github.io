
> [Hfendou.Blog](http://hfendou.gitbub.io) borned on 2018-08-02!<br><br>
> 日后将会持续地把学习历程及重要的信息记录这里！


# 多线程异常处理/线程取消/临时变量/线程安全

``` c#

 try
            {
                TaskFactory taskFactory = new TaskFactory();
                List<Task> taskList = new List<Task>();

#region 异常处理
                //多线程的委托是不允许异常的， try catch包住，写下日志
                for (int i = 0; i < 20; i++)
                {
                    string name = string.Format("btnThreadCore_Click{0}", i);
                    Action<object> act = t =>
                    {
                        try
                        {
                            Thread.Sleep(2000);
                            if (t.ToString().Equals("btnThreadCore_Click11"))
                            {
                                throw new Exception(string.Format("{0} 执行失败", t));
                            }
                            if (t.ToString().Equals("btnThreadCore_Click12"))
                            {
                                throw new Exception(string.Format("{0} 执行失败", t));
                            }
                            Console.WriteLine("{0} 执行成功", t);
                        }
                        catch (Exception ex)
                        {
                            Console.WriteLine(ex.Message);
                        }
                    };
                    taskList.Add(taskFactory.StartNew(act, name));
                }
                Task.WaitAll(taskList.ToArray());
#endregion


#region 线程取消
                ////Task不能主动取消，只能通过信号量检查的方式
                CancellationTokenSource cts = new CancellationTokenSource();
    
                for (int i = 0; i < 40; i++)
                {
                    string name = string.Format("btnThreadCore_Click{0}", i);
                    Action<object> act = t =>
                    {
                        try
                        {
                            //if (cts.IsCancellationRequested)
                            //{
                            //    Console.WriteLine("{0} 取消一个任务的执行", t);
                            //}
                            Thread.Sleep(2000);
                            if (t.ToString().Equals("btnThreadCore_Click11"))
                            {
                                throw new Exception(string.Format("{0} 执行失败", t));
                            }
                            if (t.ToString().Equals("btnThreadCore_Click12"))
                            {
                                throw new Exception(string.Format("{0} 执行失败", t));
                            }
                            if (cts.IsCancellationRequested)
                            {
                                Console.WriteLine("{0} 放弃执行", t);
                            }
                            else
                            {
                                Console.WriteLine("{0} 执行成功", t);
                            }
                        }
                        catch (Exception ex)
                        {
                            cts.Cancel();
                            Console.WriteLine(ex.Message);
                        }
                    };
                    taskList.Add(taskFactory.StartNew(act, name, cts.Token));//没有启动的任务  在Cancel后放弃启动
                }
                Task.WaitAll(taskList.ToArray());
#endregion
    

 #region 多线程临时变量
                for (int i = 0; i < 5; i++)
                {
                    //int k = i;
                    new Action(() =>
                    {
                        //Thread.Sleep(100);
                        Console.WriteLine(i);
                        //Console.WriteLine(k);
                    }).BeginInvoke(null, null);
                }
    
                foreach (var item in collection)
                {
                    //做了事件的注册  每次用的都是item
                }
    
#endregion
    

#region 线程安全 lock
                for (int i = 0; i < 10000; i++)
                {
                    int newI = i;
                    taskList.Add(taskFactory.StartNew(() =>
                        {
                            int k = 2;
                            int w = 3;
                            int iResult = k + w;
                            lock (btnThreadCore_Click_Lock)
                            {
                                this.TotalCount += 1;
                                IntList.Add(newI);
                            }
                        }));
                }
                Task.WaitAll(taskList.ToArray());
    
                Console.WriteLine(this.TotalCount);
                Console.WriteLine(IntList.Count());

#endregion

            }
            catch (AggregateException aex)
            {
                foreach (var item in aex.InnerExceptions)
                {
                    Console.WriteLine(item.Message);
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }

```