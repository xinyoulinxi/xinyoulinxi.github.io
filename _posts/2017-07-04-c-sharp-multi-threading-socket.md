---
layout: post
author: "YL"
title:  "C# 多线程学习（三）"
subtitle: "Socket 服务器与客户端通信"
date:  2017-07-04 13:16:13
tags:
    - thread
    - c#
catalog: false
header-style: text
---
# C# 多线程学习（三）Socket
正好最近用到了Socket实现了一个大小屏幕互动的应用，所以就写一下关于多线程和Socket合作编写的服务器客户端的基础教学吧。
以下分别为服务器和客户端的简单demo，分成两个C#控制台程序编译运行就可以直接互相通信了。注意端口绑定一定要一致。


**以下为服务器端的代码：**
---------------

```
using System.Net;
using System.Net.Sockets;
using System;
using System.Text;
using System.Threading;
namespace SocketServer
{
    class Program
    {
        private static byte[] result = new byte[1024];
        private static int myProt = 59999;   //端口  
        static Socket serverSocket;
        static void Main()
        {
            //服务器IP地址  ，127.0.0.1 为本机IP地址
            IPAddress ip = IPAddress.Parse("127.0.0.1");
            serverSocket = new Socket(
                AddressFamily.InterNetwork, 
                SocketType.Stream,
                ProtocolType.Tcp);

            serverSocket.Bind(new IPEndPoint(ip, myProt));  //绑定IP地址：端口  
            serverSocket.Listen(10);    //最多10个连接请求  
            Console.WriteLine("creat service {0} success", 
                serverSocket.LocalEndPoint.ToString());
            
            Thread myThread = new Thread(ListenClientConnect);
            myThread.Start();
            Console.ReadLine();
        }

        // 监听客户端是否连接  
        private static void ListenClientConnect()
        {
            while (true)
            {
                Socket clientSocket = serverSocket.Accept();
                clientSocket.Send(Encoding.ASCII.GetBytes("Server Say Hello"));
                Thread receiveThread = new Thread(ReceiveMessage);
                receiveThread.Start(clientSocket);
            }
        }

        //开启线程接收数据
        private static void ReceiveMessage(object clientSocket)
        {
            Socket myClientSocket = (Socket)clientSocket;
            while (true)
            {
                try
                {
                    //接收数据  
                    int receiveNumber = myClientSocket.Receive(result);
                    Console.WriteLine("client say : {0} " ,Encoding.ASCII.GetString(result, 0, receiveNumber));
                }
                catch (Exception ex)
                {
                    Console.WriteLine(" A client break");
                    myClientSocket.Shutdown(SocketShutdown.Both);
                    break;
                }
            }
        }
    }
}
```

以下为客户端的代码：

```
using System.Net;
using System.Net.Sockets;
using System;
using System.Text;
namespace SocketClient
{
    class Program
    {
        private static byte[] result = new byte[1024];
        static void Main(string[] args)
        {
            //设定服务器IP地址  
            IPAddress ip = IPAddress.Parse("127.0.0.1");//本地IP地址
            Socket clientSocket = new Socket(
                AddressFamily.InterNetwork,
                SocketType.Stream,
                ProtocolType.Tcp);
            try
            {
                clientSocket.Connect(new IPEndPoint(ip, 59999)); //配置服务器IP与端口 ，并且尝试连接
                Console.WriteLine("Connect Success");
            }
            catch
            {
                Console.WriteLine("Connect error");
                return;
            }
            
            int receiveLength = clientSocket.Receive(result);//接收回复，成功则说明已经接通
            Console.WriteLine("start");
            while (true)
            {//开始发送数据
                try
                {
                    string sendMessage = Console.ReadLine();
                    clientSocket.Send(Encoding.ASCII.GetBytes(sendMessage));//传送信息
                }
                catch
                {
                    clientSocket.Shutdown(SocketShutdown.Both);
                    break;
                }
            }
            Console.WriteLine("over");
            Console.ReadLine();
        }
    }
}

```
以上代码已经在我的电脑上能够直接运行了。方法如下：
将以上代码分成两个工程，然后先打开服务器程序，然后再打开客户端程序，在客户端控制台上输入，就可以在服务器端进行接收。以下为演示界面：
![这里写图片描述](/imgs/post/csharp1.png)
然后就可以进行交互了，还有更多有意思的操作，我就不多写了，毕竟主要是介绍多线程

**127.0.0.1是你本地电脑的IP地址**，如果想要和其他电脑进行通信，就将服务器端的IP改成路由器分配给你的IP地址，或者自己通过**CMD控制台**输入 **ipconfig** 进行查看，如下：
![这里写图片描述](/imgs/post/csharp2.png)

通俗点说，**就是客户端通过IP找到服务器的地址，再通过端口找到服务器的进程进行通信。所以我们会给服务器端进程分配一个端口，帮助客户端找到它。**

> 计算机中的端口数为65536，差不多就是16的地址长度，1-20000的端口一般是给系统调用的，50000以上的一般为空闲端口。



