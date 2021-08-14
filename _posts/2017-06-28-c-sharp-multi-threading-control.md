---
layout: post
author: "YL"
title:  "C# 多线程学习（二）"
subtitle: "线程操作"
date:  2017-06-28 22:00:27
tags:
    - thread
    - c#
catalog: false
header-style: text
---
# C# 多线程学习（二）线程操作
在C#创建一个线程的时候，通常使用Thread类，然后提供一个线程入口，`Thread thread =new Thread(functionName)`，线程的入口通过`ThreadStart`代理（delegate）来提供，你可以把`ThreadStart` 当作一个函数指针，指向线程所要执行的函数（或者说 **方法**），当调用`Thread.Start()`方法后，线程就开始执行`ThreadStart`所代表或者说指向的函数。 

然后我们开始编写一个简单的C#控制台应用程序来对一个创建的线程进行控制吧：

```
using System;
using System.Threading;
namespace ThreadControl
{
    class Program
    {
        public class test
        {
            public void testThread()
            {
                while (true)
                {
                    Console.WriteLine("now ,it is in child thread");
                }
            }
        }
        static void Main(string[] args)
        {

            test fun = new test();
            //下面创建一个线程，让其做test.testThread()这个任务
            Thread thread = new Thread(new ThreadStart(fun.testThread));
            thread.Start();//开始线程
            thread.Join();//让主线程等待子线程thread执行完毕
            try
            {
                Console.WriteLine("Try to restart thread");
                thread.Start();
            }
            catch (ThreadStateException)
            {
                Console.WriteLine("ThreadStateException trying to restart Alpha.Beta.Error ");
                Console.ReadLine();
            }
            Console.ReadLine();
        }
    }
}
```

**主线程Main()函数**

所有线程都是依附于Main()函数所在的线程的，Main()函数是C#程序的入口，起始线程可以称之为主线程。
如果所有的前台线程都停止了，那么主线程可以终止，而所有的后台线程都将无条件终止。
**ThreadState 属性的取值如下：**

 - Aborted：线程已停止； 
 -  AbortRequested：线程的Thread.Abort()方法已被调用，但是线程还未停止； 
 -   Background：线程在后台执行，与属性Thread.IsBackground有关；  
 - StopRequested：线程正在被要求停止； 
 - Suspended：线程已经被挂起（此状态下，可以通过调用Resume()方法重新运行
 - SuspendRequested：线程正在要求被挂起，但是未来得及响应； 
 - Unstarted：未调用Thread.Start()开始线程的运行； 


**线程的优先级**

> **当线程之间争夺CPU时间时，CPU 是按照线程的优先级给予服务的。**在C#应用程序中，用户可以设定5个不同的优先级，由高到低分别是`Highest`，`AboveNormal`，`Normal`，`BelowNormal`，`Lowest`，在创建线程时如果不指定优先级，那么系统默认为`ThreadPriority.Normal。`
> 给一个线程指定优先级，我们可以使用如下代码： //设定优先级为最低
> myThread.Priority=ThreadPriority.Lowest;
> 
> **通过设定线程的优先级，我们可以安排一些相对重要的线程优先执行，例如对用户的响应等等。**
