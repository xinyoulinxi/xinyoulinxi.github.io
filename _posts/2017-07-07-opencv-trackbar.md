---
layout: post
author: "YL"
title:  "OpenCv基础（四）"
subtitle: "Trackbar（轨迹条）的创建和使用"
date:  2017-07-07 23:01:21
tags:
    - opencv
    - c++
header-mask: 0.2
header-img: "imgs/OpenCV_logo.png"
---
createTrackbar这个函数，它创建一个可以调整数值的轨迹条，并将轨迹条附加到指定的窗口上。
函数原型如下：

```
int createTrackbar(conststring& trackbarname, conststring& winname, int* value, int count, TrackbarCallback onChange=0,void* userdata=0);  
```
下面看一下各个参数的作用：

> **第一个参数**，const string&类型的trackbarname，表示轨迹条的名字，用来代表我们创建的轨迹条。 
> **第二个参数**，const string&类型的winname，填窗口的名字，表示这个轨迹条会依附到哪个窗口上，即对应namedWindow（）创建窗口时填的某一个窗口名。
> **第三个参数**，int* 类型的value，一个指向整型的指针，表示滑块的位置。并且在创建时，滑块的初始位置就是该变量当前的值。
> **第四个参数**，int类型的count，表示滑块可以达到的**最大位置**的值。PS:滑块最小的位置的值始终为0。
> **第五个参数**，TrackbarCallback类型的onChange，首先注意他有默认值0。这是一个指向回调函数的指针，**每次滑块位置改变时，这个函数都会进行回调**。并且这个函数的原型必须为void
> **XXXX(int,void*)**;其中第一个参数是轨迹条的位置，第二个参数是用户数据（看下面的第六个参数）。如果回调是NULL指针，表示没有回调函数的调用，仅第三个参数value有变化。
> **第六个参数**，void*类型的userdata，他也有默认值0。这个参数是用户传给回调函数的数据，用来处理轨迹条事件。如果使用的第三个参数value实参是全局变量的话，完全可以不去管这个userdata参数。
> 
> 这个createTrackbar函数，为我们创建一个具有特定名称和范围的轨迹条（Trackbar，或者说是滑块范围控制工具），指定一个和轨迹条位置同步的变量。而且要指定回调函数onChange（第五个参数），**在轨迹条位置改变的时候来调用这个回调函数。**并且我们知道，创建的轨迹条显示在指定的winname（第二个参数）所代表的窗口上。

下面是一个完整的实例代码——用一个滑动条来调节两张同样大小的图片的混合程度，代码如下：

```


#include <opencv2/core/core.hpp>  
#include<opencv2/highgui/highgui.hpp>  
#include <iostream>  

   
using namespace std;
using namespace cv;

 //回调函数的声明
static void CreatWindowsAndControl(int, void *);

//原始图像，大小相同
Mat srcImage_1;
Mat srcImage_2;
//窗口名字
char * windows_name = "showImg_Windows";

//滚动条的滑块代表的value
static int weight = 0;

//开始函数
int main()
{
	srcImage_1 = imread("src.jpg");
	srcImage_2 = imread("src_1.jpg");


	//创建窗口  
	namedWindow(windows_name);

	//创建轨迹条  
	createTrackbar("混合程度：", windows_name, &weight, 100, CreatWindowsAndControl);
	CreatWindowsAndControl(weight, 0);

	waitKey(0);
	return 0;
}


static void CreatWindowsAndControl(int, void *)
{
	//以下为将滑块的value转换为addWeighted（）函数的适应值
	double d_weight = (double)weight;
	d_weight /= 100.0;
	//end
	//输出的混合图像
	Mat out;
	addWeighted(srcImage_1, d_weight, srcImage_2, 1.0 - d_weight, 0.0, out);
	imshow(windows_name, out);
}
```

运行界面：
-----

![这里写图片描述](/imgs/post/opencv/4-1.png)




> PS:一定注意原始的两张图像大小是一样的。