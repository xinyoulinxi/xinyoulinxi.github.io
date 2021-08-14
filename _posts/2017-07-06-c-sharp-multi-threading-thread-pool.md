---
layout: post
author: "ylvoid"
title:  "C# 多线程学习（六）"
subtitle: "线程池（ThreadPool）——线程资源的复用和自动管理"
date:  2017-07-06 22:33:14
tags:
    - thread
    - c#
catalog: false
header-style: text
---
什么是线程池
------

大家都知道，我们在打开一个应用的时候，操作系统是要做很多的事情的，动态链接、装载、分配虚拟空间、等等等等，其实一个应用的打开同时也伴随着一个进程的建立。

**进程的建立是需要时间的**，在进程上开线程也是需要消耗CPU时间,**操作系统需要分配给新开的线程地址空间、栈空间、寄存器等，在线程结束的时候，操作系统又将这些东西回收（着同样需要消耗时间）。**

  所以我们在多线程的处理中如果遇到要很多次地开启线程去处理一些很简单但是要求应答速度快的任务。我们就可以把以前用过的线程拿出来重新使用，而不用每次都创建分别那些地址空间和寄存器。所以我们的**任务如果对线程创建的效率要求很高的话**（比如服务器对客户端的应答），就可以考虑使用线程池。
  


----------
C#已经在自己的库中实现了内存池，其命名空间如下：

```
using System.Threading;

```

> 然后你无需自己建立线程，只需把你要做的工作写成函数，然后作为参数传递给ThreadPool.QueueUserWorkItem()方法就行了，传递的方法就是依靠WaitCallback代理对象，而线程的建立、管理、运行等工作都是由系统自动完成的，你无须考虑那些复杂的细节问题。

听起来很棒吧，但是在使用线程池之前，我们要先了解一下线程池的好处和缺陷

**缺陷**

>  - ThreadPool类提供一个由系统维护的线程池（可以看作一个线程的容器），该容器需要 Windows 2000 以上系统支持，因为其中某些方法调用了只有高版本的Windows才有的API函数。
>  - ThreadPool类是一个静态的类，你不能也不需要生成它的对象。而且一旦使用该方法在线程池中添加了一个项目，那么该项目将是无法取消的。
>  - 你无法对线程池中的线程进行操作

 


 **好处**
 

>你无需自己建立线程，只需把你要做的工作写成函数，然后作为参数传递给ThreadPool.QueueUserWorkItem()方法就行了，传递的方法就是依靠WaitCallback代理对象，而线程的建立、管理、运行等工作都是由系统自动完成的，你无须考虑那些复杂的细节问题。

如果你理解了线程池的好处和坏处还是决定去使用，以下为一个示例代码：

```
using System;
using System.Threading;

namespace threadTest
{
    class Program
    {
       public delegate void func(object ob);
       public object objLock=new object();
         public void  Run(object c)
        {
            int num;
            int.TryParse(c.ToString(), out num);
            for (int i = 0; i < num; i++)
            {
                lock (objLock)
                {
                    Console.WriteLine("now num is {0} print {1} ", num, i);
                }
            }
        }
        static public void Main()
        {
           
            bool isOk = false;
            Program p = new Program();
            try
            {
                 //加入线程池，执行这个任务
                ThreadPool.QueueUserWorkItem(new WaitCallback(p.Run), 0);
                isOk = true;

            }
            catch (NotSupportedException e)
            {
                Console.WriteLine("this API is not running on non-Windows 2000 system");
                isOk = false;
            }
            if (isOk)//如果当前系统支持ThreadPool
            {
                for (int i = 0; i < 9; i++)
                {
                    ThreadPool.QueueUserWorkItem(new WaitCallback(p.Run), i);
                }
            }
            Console.ReadKey();
        }
    }
}
```

上面的ObjectLock 为一个object类，专为多线程程序而存在的，它提供了一些有用的原子操作。

> 原子操作：就是在多线程程序中，如果这个线程调用这个操作修改一个变量，那么其他线程就不能修改这个变量了，这跟lock关键字在本质上是一样的。

传递的那个object 也可以被其他的数据结构所替代，可以包含很多的数据在里面，从而进行很复杂·的参数传递操作。

好了，ThreadPool已经差不多完成了，就此完结吧。C#只是做一下了解，以后还是继续深入了解C++。