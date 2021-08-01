---
layout: post
title:  "光流金字塔calcOpticalFlowPyrLK进行特征点跟踪"
date:    2018-01-26 19:42:30
tags:
    - 关键点检测和追踪
    - ar
author: "YL"
subtitle: ""
---
# 光流金字塔calcOpticalFlowPyrLK进行特征点跟踪
> 光流描述的是图像上每个像素点的灰度的位置（速度）变化情况，光流的研究是利用图像序列中的像素强度数据的时域变化和相关性来确定各自像素位置的“运动”。研究光流场的目的就是为了从图片序列中近似得到不能直接得到的运动场。
> 光流法的前提假设：
> - （1）相邻帧之间的亮度恒定；
> - （2）相邻视频帧的取帧时间连续，或者，相邻帧之间物体的运动比较“微小”；
> - （3）保持空间一致性；即，同一子图像的像素点具有相同的运动；

光流金字塔`calcOpticalFlowPyrLK`的C++定义如下：

```
CV_EXPORTS_W void calcOpticalFlowPyrLK( InputArray prevImg, InputArray nextImg,
                           InputArray prevPts, CV_OUT InputOutputArray nextPts,
                           OutputArray status, OutputArray err,
                           Size winSize=Size(21,21), int maxLevel=3,
                           TermCriteria criteria=TermCriteria(
                            TermCriteria::COUNT+TermCriteria::EPS,
                            30, 0.01),
                           double derivLambda=0.5,
                           int flags=0,
                           double minEigThreshold=1e-4);
```

各个参数代表的含义如下：
- `prevImg` 
你的标定图像的灰度图
- `nextImg`
你想搜寻的图像的灰度图
- `prevPts` 
输入的标定图像的特征点（可以是其他特征点检测方法找到的点）
- `nextPts` 
输出场景的特征点
- `status` 
输出状态向量（无符号`char`），如果在当前图像中能够光流得到标定的特征点位置改变，则设置`status`的对应位置为1，否则设置为0
- `err `
输出错误向量；向量的每个元素被设为相应特征的一个错误，误差测量的类型可以在flags参数中设置；如果流不被发现然后错误未被定义（使用`status`（状态）参数找到此情形）。
- `winSize` 
在每个金字塔水平搜寻窗口的尺寸
- `maxLevel`
金字塔的高度，初始为3层

当使用`calcOpticalFlowPyrLK`作为光流金字塔的算法时候，我们只需要知道以下的几点：

> -  `calcOpticalFlowPyrLK`必须和其他的角点识别算法进行搭配使用，比如我这里使用的`goodFeaturesToTrack`，将其他的角点识别算法中获得的角点作为光流算法的`prevPts`
> -  `status` 的大小和当前需要识别的光流移动的特征点大小一样，所以我们可以判定当前的图像是否还能与标定图像进行光流的依据


**运行环境为：**
-  opencv 2.3.1
- 
-  vs2017
-
以下是一个傻瓜示例代码：

```

#include<iostream>  

#include<opencv.hpp>


using namespace std;
using namespace cv;


int main()
{
	Mat image1, image2;
	vector<Point2f> point1, point2, pointCopy;
	vector<uchar> status;
	vector<float> err;


	VideoCapture video(0);
	video >> image1;
	Mat image1Gray, image2Gray;
	cvtColor(image1, image1Gray, CV_RGB2GRAY);
	goodFeaturesToTrack(image1Gray, point1, 100, 0.01, 10, Mat());
	pointCopy = point1;
	for (int i = 0; i < point1.size(); i++)    //绘制特征点位  
	{
		circle(image1, point1[i], 1, Scalar(0, 0, 255), 2);
	}
	namedWindow("光流特征图");
	while (true)
	{
		video >> image2;
		if (waitKey(33) == ' ')  //按下空格选择当前画面作为标定图像  
		{
			cvtColor(image2, image1Gray, CV_RGB2GRAY);
			goodFeaturesToTrack(image1Gray, point1, 100, 0.01, 10, Mat());
			pointCopy = point1;
		}
		cvtColor(image2, image2Gray, CV_RGB2GRAY);
		calcOpticalFlowPyrLK(image1Gray, image2Gray, point1, point2, status, err, Size(50, 50), 3); //LK金字塔       
		int tr_num = 0;
		vector<unsigned char>::iterator status_itr = status.begin();
		while (status_itr != status.end()) {
			if (*status_itr > 0)
				tr_num++;
			status_itr++;
		}
		if (tr_num < 6) {
			cout << "you need to change the feat-img because the background-img was all changed" << endl;
			if (waitKey(0) == ' ') {
				cvtColor(image2, image1Gray, CV_RGB2GRAY);
				goodFeaturesToTrack(image1Gray, point1, 100, 0.01, 10, Mat());
				pointCopy = point1;
			}
		}
		for (int i = 0; i < point2.size(); i++)
		{
			circle(image2, point2[i], 1, Scalar(0, 0, 255), 2);
			line(image2, pointCopy[i], point2[i], Scalar(255, 0, 0), 2);
		}

		imshow("光流特征图", image2);
		swap(point1, point2);
		image1Gray = image2Gray.clone();
	}
	return 0;
}
```

效果如图：
- 标定图像
- 
![这里写图片描述](/imgs/post/LK-track-1.png)
- 镜头移动
- 
![这里写图片描述](/imgs/post/LK-track-2.png)

ps: 红点为标定点，蓝线为跟踪点的光流移动