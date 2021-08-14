---
layout: post
author: "YL"
title:  "OpenCv基础（五)"
subtitle: "摄像头画面获取和输出"
date:  2017-07-08 15:04:29
tags:
    - opencv
    - c++
header-mask: 0.2
header-img: "imgs/OpenCV_logo.png"
---
直接贴代码吧，只是一个小的测试，以下为代码：

```




#include <opencv2/core/core.hpp>  
#include<opencv2/highgui/highgui.hpp>  
#include <iostream>  
using namespace std;
using namespace cv;


int main()
{

	VideoCapture camera(0);//初始化摄像头，0代表默认摄像头

	Mat frame;

	while (true) 
	{
		if (!camera.isOpened()) 
		{
			cout << "open camera error" << endl;
			break;
		}

		camera >> frame;//获取当前帧
		
		imshow("Frame_Windows", frame);
		
		waitKey(33);//每秒显示30多帧，相当于 30+ fps
		
	}
	waitKey(0);
	return 0;
}


```

还有一点说明：

> VideoCapture类有两种用法，一种是VideoCapture(const string& filename)用来打开视频文件，一种是VideoCapture(int device)用来打开设备。


 