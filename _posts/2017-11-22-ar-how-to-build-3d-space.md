---
layout: post
author: "ylvoid"
subtitle: ""
title:  "【AR】（一）如何通过捕捉特征图像来建立起三维空间"
date:   2017-11-22 20:58:24
header-img: /imgs/page_bg.jpg
tags:
    - AR
    - c++
    - CV
header-img: "imgs/ar-bg.jpg"
header-mask: 0.5
---

# 【AR】（一）如何通过捕捉特征图像来建立起三维空间
在AR的技术中，最主要的技术点主要有三个：

 

    1. 捕获特征图像
    2. 跟踪特征图像的移动
    3. 计算捕获的特征图像相对于原特征图像的偏移向量

为什么说这三个是最重要的呢
原因就是AR技术最关键的就是能够高效的捕捉你给予的特征图像，并在此之上将二维的视频图像转换成三维的空间，然后在此之上展示你想要展示的3D模型。



捕获特征图像
======

说到特征匹配，就得先说一下什么是特征点，现在几乎所有的特征图像捕捉算法都是基于特征点进行匹配的

特征点，顾名思义，就是图像中或者全局或者局部的一种能够描述图片特征的具有标识性的点。
一般情况下，**特征点是指的是图像灰度值发生剧烈变化的点或者在图像边缘上曲率较大的点**(即两个边缘的交点)。听起来有点像是角点，但是实际是角点只是特征点中的一类。
举一个例子，对于FAST特征点检测算法，特征点判别如下：
判别特征点p是否是一个特征点，可以通过判断以该点为中心画圆，该圆过16个像素点。设在圆周上的16个像素点中是否最少有n个连续的像素点满足都比Ip+t大，或者都比Ip−t小。（这里Ip指的点p的灰度值，t是一个阈值）
如果满足这样的要求，则判断p是一个特征点，否则p不是。


![这里写图片描述](/imgs/post/ar/ar1.png)

通过这些点我们可以用来识别图像、进行图像配准、进行3D重建等。

由于我们这里只是讲解AR的关键技术，所以就不对特征匹配算法进行一一地讲解了
我们梳理一下一般的特征图像匹配算法的流程：

     PS：这里设摄像头图像为srcImg，想捕获的图像为grabImg

 1. 灰度化原图像srcImg
 2. 进行平滑滤波，去除**噪点**
 3. 特征点检测，取出grabImg在srcImg上匹配的特征点（一般采用FAST算法）
 4. 筛选特征点，去除特征点中的**坏点**（一般采用Lowe's算法来进一步获取优秀匹配点）
 5. 匹配对应的特征点
 6. 求出srcImg上的grabImg的偏移和变换向量
![ 特征图像匹配的流程图](/imgs/post/ar/ar2.png)

 最后得到的效果大致如下（**网络图片，特征点被用颜色圈标出，并用线条一一对应**）：
 ![这里写图片描述](/imgs/post/ar/ar3.png)

建立空间坐标系
=======

我们通过图像特征匹配的方式可以得到一些什么东西——**可以得到我们的特征图像和从摄像图像中捕捉到的图像之间的对应关系**
这种对应关系有什么用呢，想一下，假设我们的普通摄像头拍到的图像如下：
![这里写图片描述](/imgs/post/ar/ar4.png)

我们想寻找的特征图像如下：

![这里写图片描述](/imgs/post/ar/ar5.png)

我们通过特征匹配算法匹配这两张图，可以得到什么信息？

第一、可以获取到特征图像在摄像头图像中的位置（**确定位置**）
第二、可以得到特征图像的大小变化（**确定大小变化**）
第三、可以得到特征图像的**旋转角度变化**（三维空间下的旋转角度）

比如上面的摄像头图像，我们可以获得这样的信息（数据是瞎编的，领会意思就行）：
![这里写图片描述](/imgs/post/ar/ar6.png)
获得了这样的信息有什么用呢？
通过这些信息我们就可以**确定特征图像的法向量方向**，**进而求出特征图像在三维空间中的平面场的延展（这个平面就是我们展示AR-3D模型的基础平面）**

也许上面的图像不太明显，我们用这样的图像进行示例（PS：下面的图像绘画也许有一定错误）

![这里写图片描述](/imgs/post/ar/ar7.png)

通过这样的方式，我们就可以建立起一个神奇的三维坐标系：

![这里写图片描述](/imgs/post/ar/ar8.png)

然后的事情就变得很简单了，将你的3d模型通过OpenGL或者其他的渲染手段，在这个不断变化的坐标系中，不停地改变其大小、旋转角然后渲染出来，或者直接就在这个平面上移动，都是可行的了。
我们所需要的，就是为这个模型在2d的摄像头图片中，找到其相对于特征图像的三维特征位置和标定

不过，如果仅仅通过这样的特征匹配算法来一帧一帧的匹配，计算量是非常非常大的，有没有办法不每次都这样全局的去寻找特征图像，而是跟踪其移动，然后建立坐标系呢？
当然有
所以下一节我们要了解的就是AR的另外一个技术点——**图像跟踪**