---
layout: post
author: "YL"
title:  "C# 多线程学习（四）"
subtitle: "带参数的线程启动"
date:  2017-07-04 16:34:56
tags:
    - thread
    - c#
catalog: false
header-style: text
---
 在很多应用下，我们想要执行一个不带有一定先决条件的任务，比如如下代码：
 

```
using System;
using System.Threading;

namespace threadTest
{
    class Program
    {
        int interval = 200;//间隔时间
        static void Main()
        {
            Program p = new Program();
            Thread thread = new Thread(new ThreadStart(p.NonParameterRun));
            thread.Start();
            Console.ReadKey();
        }
        /// <CodeDoctor>  
        /// 不带参数的启动方法  
        /// <CodeDoctor>  
        public void NonParameterRun()
        {
            for (int i = 0; i < 10; i++)
            {
                Console.WriteLine("now i = {0}", i);
                Thread.Sleep(interval);
            }
        }
    }
}
```

这个任务就很简单，没有参数，参数在全局变量中设置好了，执行每隔 interval ms 输出一次 i 的值，但是这种方式的局限性很大，比如如果有多个任务要执行不同时间间隔的输出就很尴尬，还好C#提供了带参数的线程启动方案，在thread.start(object),由于C#中所有的基础类都继承于object类，所有我们可以通过强制类型转换达到我们想传入的不同类型的参数。以下为一个带参的例子：

```
using System;
using System.Threading;

namespace threadTest
{
    class Program
    {
        static void Main()
        {
            Program p = new Program();
            Thread thread1 = new Thread(p.NonParameterRun);
            Thread thread2 = new Thread(p.NonParameterRun);
            thread1.Start(10);
            thread2.Start(20);
            Console.ReadKey();
        }
        /// <CodeDoctor>  
        /// 带参数的启动方法  
        /// <CodeDoctor>  
        public void NonParameterRun(object interval)
        {
            int ms;
            int.TryParse(interval.ToString(), out ms);//用TryParse进行类型转换  
            for (int i = 0; i < 10; i++)
            {
                Console.WriteLine("now i = {0}", i);
                Thread.Sleep(ms);
            }
        }
    }
}
```
**继续思考**

上面解决了实现带参的thread任务，当然我们有时候也会想传入多个参数，比如上面的任务，我们不仅想传入休眠时间，还想传入输出的 i 的数量
，我们当然可以通过传入数组、类之类的想法来实现，但那样就太过于麻烦

**所以我们可以通过构造一个类来实现，代码如下：**

```
using System;
using System.Threading;

namespace threadTest
{
    class Program
    {
        class myThread
        {
            private int m_interval;
            private int m_num;
            Thread m_thread;
            public myThread(int ms,int num)
            {
                m_interval = ms;
                m_num = num;
                m_thread = new Thread(new ThreadStart(this.Task));
            }
            public void Start()
            {
                if (m_thread != null)
                {
                    m_thread.Start();
                }
                
            }
            /// <CodeDoctor>  
            /// 带多个参数的启动方法  
            /// <CodeDoctor>  
            public void Task()
            {
                for (int i = 0; i < m_num; i++)
                {
                    Console.WriteLine("now i = {0}", i);
                    Thread.Sleep(m_interval);
                }
            }
        }
        static void Main()
        {
            myThread thread1 = new myThread(10,20);
            myThread thread2 = new myThread(15,25);
            thread1.Start();
            thread2.Start();
            Console.ReadKey();
        }
    }
}
```

这样我们就可以在自己的thread中构建不同的参数结构或者实现很多其他的功能


----------

我想大家已经了解了如何进行参数的传递和处理，而且也了解了如何封装一个自己的多参数或者多任务的thread类。
很多时候，虽然C#、.net的类库没有给我们提供这些实现，但只要想方法，还是能自己解决的。