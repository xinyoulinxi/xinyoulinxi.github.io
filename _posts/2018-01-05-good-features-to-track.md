---
layout: post
title:  "goodFeaturesToTrack关键点检测"
date:    2018-01-05 19:42:30
tags:
    - 关键点检测和追踪
    - CV
    - 技术细节
author: "ylvoid"
subtitle: ""
catalog: false
header-style: text
---

环境：`opencv 2.3.1`

函数 `goodFeaturesToTrack`，函数原型：
```
void goodFeaturesToTrack( InputArray image, OutputArray corners,  
                                     int maxCorners, double qualityLevel, double minDistance,  
                                     InputArray mask=noArray(), int blockSize=3,  
                                     bool useHarrisDetector=false, double k=0.04 );  
```
- 第一个参数`image`：8位或32位单通道灰度图像；
- 第二个参数`corners`：位置点向量，保存的是检测到的角点的坐标；
- 第三个参数`maxCorners`：定义可以检测到的角点的数量的最大值；
- 第四个参数`qualityLevel`：检测到的角点的质量等级，角点特征值小于`qualityLevel` 乘以 最大特征值的点将被舍弃；
- 第五个参数`minDistance`：两个角点间最小间距，以像素为单位；
- 第六个参数`mask`：指定检测区域，若检测整幅图像，`mask`置为空`Mat`；
- 第七个参数`blockSize`：计算协方差矩阵时窗口大小；
- 第八个参数`useHarrisDetector`：是否使用`Harris`角点检测，为false，则使用`Shi-Tomasi`算子；
- 第九个参数k：留给`Harris`角点检测算子用的中间参数，一般取经验值`0.04~0.06`。第八个参数为`false`时，该参数不起作用；


函数使用实例：

```
int main()
{
	Mat img = imread("test.jpg");
	Mat grayImage;
	//转换灰度图
	cvtColor(img, grayImage, CV_BGR2GRAY);

	//开始进行角点检测  
	vector<Point2f> dstPoint2f;
	goodFeaturesToTrack(grayImage, dstPoint2f, 200, 0.01, 10, Mat(), 3);

	//遍历每个点，进行绘制，便于显示  
	Mat dstImage;
	img.copyTo(dstImage);
	for (int i = 0; i < (int)dstPoint2f.size(); i++)
	{
		circle(dstImage, dstPoint2f[i], 3, Scalar(theRNG().uniform(0, 255), theRNG().uniform(0, 255), theRNG().uniform(0, 255))
			, 2, 8);
	}
	imshow("Frame_Windows", dstImage);
	waitKey();
	return 0;
}
```
运行结果如下
![](/imgs/post/feature_1.png)