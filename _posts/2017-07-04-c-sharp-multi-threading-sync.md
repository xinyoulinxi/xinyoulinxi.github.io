---
layout: post
author: "YL"
title:  "C# 多线程学习（五）"
subtitle: "线程同步和冲突解决"
date:  2017-07-04 22:04:30
tags:
    - thread
    - c#
catalog: false
header-style: text
---
**首先先说一个线程不同步的例子吧**，以下为售票员的模拟售票，多个售票员出售100张门票，代码如下：

```
using System;
using System.Text;
using System.Collections.Generic;
using System.Threading;

namespace threadTest
{
    class Program
    {

        class ThreadLock
        {
            private Thread thread_1;
            private Thread thread_2;

            private List<int> tickets;

             private object objLock = new object();//对象锁的对象
            public ThreadLock()
            {
                thread_1 = new Thread(Run);
                thread_1.Name = "Sailer_1";
                thread_2 = new Thread(Run);
                thread_2.Name = "Sailer_2";
            }
            public void Start()
            {
                tickets = new List<int>(100);
                for(int i = 1; i <= 100; i++)
                {
                    tickets.Add(i);
                }
                thread_1.Start();
                thread_2.Start();
            }
            public void Run()
            {
                while (tickets.Count > 0)
                {
                   
                        int get = tickets[0];
                        Console.WriteLine("{0} sail a ticket ,ticket number :{1} ",
                            Thread.CurrentThread.Name, get.ToString());
                        tickets.RemoveAt(0);
                        Thread.Sleep(1);

                }
            }
        }
        static void Main()
        {
            ThreadLock TK = new ThreadLock();
            TK.Start();
            Console.ReadKey();
        }
    }
}
```

以上为一个模拟售票系统，两个线程对一个票箱进行操作，每次都取出最上层的票，然后输出，运行之后查看结果会发现在在同一张票上，两个线程都可能同时卖出，如下：
![这里写图片描述](https://img-blog.csdnimg.cn/img_convert/8910acdc25cac14ca2aba9a61bda9d58.png)

出现以上的情况的原因就是在多线程的的情况之下，线程的执行顺序是不可控的，就可能会出现以上的情况，具体原因可能如下：

> 
> 请看代码：
```
                        int get = tickets[0];
                        Console.WriteLine("{0} sail a ticket ,ticket number :{1} ",
                        Thread.CurrentThread.Name, get.ToString());
                        tickets.RemoveAt(0);
```

> 
> 比如，线程A在刚从tickets中确定要取最底下的一张票之后还未将这张票输出并删除，这时候线程A被分配的CPU时间就用光了。  
> 
> 然后轮到另一个线程B执行，线程B的时间充足，也同样确认了线程A刚才确定的那张票，然后取出了那张票，取出然后输出并删除掉那张票，然后将CPU控制权交到了线程A上。
> 
> 又轮到了线程A执行，线程A由于刚才已经确定了选定的票号，所以直接输出了那个票号，然后将最底下的票删除。所以可以看到取票有一段是跳跃着取得，如：1，3，5，7，...

线程同步
-----

> 出现这种情况的原因就是多个线程都是对同一个资源进行操作所致，所以在多线程编程应尽可能避免这种情况，当然有些情况下确实避免不了这种情况，这就需要对其采用一些手段来确保不会出现这种情况，这就是所谓的**线程的同步**。
> 在C#中实现线程的同步有几种方法：**lock、Mutex、Monitor、Semaphore、Interlocked**和**ReaderWriterLock**等。同步策略也可以分为同步上下文、同步代码区、手动同步几种方式。

Lock同步
------
针对上面的代码，要保证不会出现混乱的情况，可以用lock关键字来实现，出现问题的部分就是在于判断剩余票数是否大于0，如果大于0则从当前总票数中减去最大的一张票，因此可以对这部分进行lock处理，代码如下：

```
            public void Run()
            {
                while (tickets.Count > 0)
                {
                    lock (objLock)
                    {
                        if (tickets.Count > 0)
                        {
                            int get = tickets[0];
                            Console.WriteLine("{0} sail a ticket ,ticket number :{1} ",
                                Thread.CurrentThread.Name, get.ToString());
                            tickets.RemoveAt(0);
                            Thread.Sleep(1);
                        }
                    }
                }
            }
```
这样处理之后，这个售票系统就变得正常了，效果如下：
![这里写图片描述](/imgs/post/csharp3.png)
总的来说，lock语句是一种有效的、不跨越多个方法的小代码块同步的做法，也就是使用lock语句只能在某个方法的部分代码之间，不能跨越方法。

Monitor类
--------
针对以上的处理方法，我们用Monitor类来处理的话是如下代码：

```
public void Run()
            {
                while (tickets.Count > 0)
                {
                    Monitor.Enter(objLock);
                    if (tickets.Count > 0)
                    {
                        int get = tickets[0];
                        Console.WriteLine("{0} sail a ticket ,ticket number :{1} ",
                        Thread.CurrentThread.Name, get.ToString());
                        tickets.RemoveAt(0);
                        Thread.Sleep(1);
                    }

                }
                Monitor.Exit(objLock);
            }
```

运行可以知道，这段代码和lock方法的结果是一样的，当然其实lock就是用Monitor类实现的，除了锁定代码区，我们还可用Monitor类的Wait（）和 pulse（）方法。

> Wait()方法是临时释放当前活得的锁，并使当前对象处于阻塞状态
> Pulse()方法是通知处于等待状态的对象可以准备就绪了，它一会就会释放锁。

**下面我们来实现一个生产者和消费者模式，生产者线程负责生产数据，消费者线程将生产者生产出来的数据输出，代码如下：**

```
using System;
using System.Text;
using System.Collections.Generic;
using System.Threading;

namespace threadTest
{
    class Program
    {
        public class Cell
        {
            int cellContents; // Cell对象里边的内容
            bool readerFlag = false; // 状态标志，为true时可以读取，为false则正在写入
            public int ReadFromCell()
            {
                lock (this) // Lock关键字保证了当前代码块在同一时间只允许一个线程进入执行
                {
                    if (!readerFlag)//如果现在不可读取
                    {
                        try
                        {
                            //等待WriteToCell方法中调用Monitor.Pulse()方法将这个线程唤醒
                            Monitor.Wait(this);
                        }
                        catch (SynchronizationLockException e)
                        {
                            Console.WriteLine(e);
                        }
                    }
                    Console.WriteLine("Use: {0}", cellContents);
                    readerFlag = false;
                    //重置readerFlag标志，表示消费行为已经完成
                    Monitor.Pulse(this);
                    //通知WriteToCell()方法（该方法在另外一个线程中执行，等待中）
                }
                return cellContents;
            }

            public void WriteToCell(int n)
            {
                lock (this)
                {
                    if (readerFlag)
                    {
                        try
                        {
                            Monitor.Wait(this);
                        }
                        catch (SynchronizationLockException e)
                        {
                            //当同步方法（指Monitor类除Enter之外的方法）在非同步的代码区被调用
                            Console.WriteLine(e);
                        }
                    }
                    cellContents = n;
                    Console.WriteLine("Produce: {0}", cellContents);
                    readerFlag = true;
                    Monitor.Pulse(this);
                    //通知另外一个线程中正在等待的ReadFromCell()方法
                }
            }
        }
        public class CellProd
        {
            Cell cell; // 被操作的Cell对象
            int quantity = 1; // 生产者生产次数，初始化为1 

            public CellProd(Cell box, int request)
            {
                cell = box;
                quantity = request;
            }
            public void ThreadRun()
            {
                for (int looper = 1; looper <= quantity; looper++)
                    cell.WriteToCell(looper); //生产者向操作对象写入信息
            }
        }

        public class CellCons
        {
            Cell cell;
            int quantity = 1;

            public CellCons(Cell box, int request)
            {
                //构造函数
                cell = box;
                quantity = request;
            }
            public void ThreadRun()
            {
                int valReturned;
                for (int looper = 1; looper <= quantity; looper++)
                    valReturned = cell.ReadFromCell();//消费者从操作对象中读取信息
            }
        }
        public static void Main(String[] args)
        {
            int result = 0; //一个标志位，如果是0表示程序没有出错，如果是1表明有错误发生
            Cell cell = new Cell();

            //下面使用cell初始化CellProd和CellCons两个类，生产和消费次数均为20次
            CellProd prod = new CellProd(cell, 20);
            CellCons cons = new CellCons(cell, 20);

            Thread producer = new Thread(new ThreadStart(prod.ThreadRun));
            Thread consumer = new Thread(new ThreadStart(cons.ThreadRun));
            //生产者线程和消费者线程都已经被创建，但是没有开始执行 
            try
            {
                producer.Start();
                consumer.Start();

                producer.Join();
                consumer.Join();
                Console.ReadLine();
            }
            catch (ThreadStateException e)
            {
                //当线程因为所处状态的原因而不能执行被请求的操作
                Console.WriteLine(e);
                result = 1;
            }
        }
    }
}
```

这个例程中，生产者线程和消费者线程是交替进行的，生产者写入一个数，消费者立即读取并输出。
同步是通过等待Monitor.Pulse()来完成的。
首先生产者生产了一个数据，而同一时刻消费者处于等待状态，直到收到生产者的“脉冲(Pulse)”通知它生产已经完成，此后消费者进入消费状态。循环往复，效果如下：
![这里写图片描述](/imgs/post/csharp4.png)
差不多如此吧，上面的方法已经可以帮助我们解决多线程中可能出现的大部分问题，剩下的就不再介绍了，同步的处理也到此结束了。