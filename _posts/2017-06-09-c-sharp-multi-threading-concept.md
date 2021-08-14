---
layout: post
author: "YL"
title:  "C# 多线程学习（一）"
subtitle: "多线程的概念"
date:  2017-06-28 21:36:00
tags:
    - thread
    - c#
catalog: false
header-style: text
---
**首先我们先谈一谈什么是进程吧**

一个可执行文件的代码可以说是一个程序，当这个程序开始执行和装载之后，其所用到的所有内存和系统资源就叫做进程
同时，一个进程又由一个或则多个线程组成
**线程又是什么呢？**
线程是程序中的一个执行流，每个线程都有自己的专有寄存器(栈指针、程序计数器等)，但代码区是和进程中的其他线程共享的，即不同的线程可以执行同样的函数
**什么是多线程？**
多线程是指程序中包含多个执行流，即在一个程序中可以同时运行多个不同的线程来执行不同的任务，也就是说单个进程创建多个**并行执行**的线程来完成不同的任务。
**多线程的好处**
可以提高CPU的利用率。在多线程程序中，一个线程必须等待（比如访问文件存储区或则进行I/O的等待）的时候，CPU可以运行其它的线程而不是放弃剩下的任务而只是等待，这样就大大提高了程序的效率。 
**多线程的不利方面**

 - 线程也是一个程序，所以线程要占用内存，线程越多占用内存也越多；
 - 多线程需要协调和管理，CPU在不同线程之间进行并行执行的时候，会进行很多单线程的情况下不会有的额外上下文切换和线程切换的开销
 - 线程之间对共享资源的访问会相互影响，要解决公用同一共享资源的问题；

**以下为一线程的示例：**

```
using System;
using System.Threading;


namespace test1
{
    class program
    {
       static  public void test()
        {
            Console.WriteLine("hello ,it is in thread");
        }
        static void Main()
        {

            Thread thread = new Thread(test);
            thread.Name = "thread1";//线程的属性
            thread.Start();//开始线程
                           //注意，此时主线程也会同步开始进行
                           //thread.Join();  //加上这句则当前线程会等待这个新开的线程结束再继续
            Console.WriteLine("it is in main thread");
            Console.ReadLine();
        }
    }


}

```
输出如下：

> hello ,it is in thread
> it is in main thread


我们通过其中提供的Thread类来创建和控制线程，ThreadPool类用于管理线程池。

**C# 多线程操作有几个很重要的方法，如下：**

```
Start()：启动线程；
Sleep(int )：暂停当前线程指定的毫秒数； 

```


 