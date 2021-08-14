---
layout: post
author: "YL"
title:  "OpenCv基础（二)"
subtitle: "图像的亮度和对比度属性的调节"
date:  2017-05-19 10:58:14
tags:
    - opencv
    - c++
header-mask: 0.2
header-img: "imgs/OpenCV_logo.png"
---
图像处理
----

一般来说，图像处理算子是带有一幅或多幅输入图像、产生一幅输出图像的函数。
图像变换可分为以下两种：

 - 点算子（像素变换） 
 - 邻域（基于区域的）算子

像素变换
----

在这一类图像处理变换中，**仅仅根据输入像素值（有时可加上某些全局信息或参数）计算相应的输出像素值。**
这类算子包括 **亮度和对比度调整** ，以及**颜色校正和变换**。

亮度和对比度调整
--------

两种常用的点过程（即点算子），是用常数对点进行 乘法 和 加法 运算：
**g(x) = alpha*f(x) + beta**

两个参数 alpha > 0 和 beta 一般称作 **增益 和 偏置 参数**。我们往往用这两个参数来分别控制 对比度 和 亮度 。
**可以把 f(x) 看成源图像像素，把 g(x) 看成输出图像像素**。这样一来，上面的式子就能写得更清楚些：
g(i,j) = alpha * f(i,j) + beta

**其中， i 和 j 表示像素位于 第i行 和 第j列 。**
测试代码如下：

```
void TestChangeAlphaAndBeta(){

	double alpha; /** 控制对比度 */
	int beta;  /** 控制亮度 */

	Mat img = imread("test1.jpg");
	Mat newImage = Mat::zeros(img.size(), img.type());
	cout << "* Enter the alpha value [1.0-3.0]: ";
	cin >> alpha;
	cout << "* Enter the beta value [0-100]: ";
	cin >> beta;
	for (int y = 0; y < img.rows; y++)
	{
		for (int x = 0; x < img.cols; x++)
		{
			for (int c = 0; c < 3; c++)
			{
				newImage.at<Vec3b>(y, x)[c] =
					saturate_cast<uchar>(alpha*(img.at<Vec3b>(y, x)[c]) + beta);
				/*
				saturate_cast原理应当如下：
				if(data<0)
				data=0;
				else if(data>255)
				data=255;
				*/
			}
		}
	}
	imshow("src1", img);
	imshow("newImage", newImage);
	waitKey(0);
}
```
程序运行结果如下**（alpha为1.5，beta为20）**：

![这里写图片描述](/imgs/post/opencv/2-1.png)

在上一节没有讲很基础的东西，这里再讲一些：

1、一上来，我们要建立两个变量，以存储用户输入的 alpha 和 beta ：

```
double alpha;
int beta;
```

2、然后，用 imread 载入图像，并将其存入一个Mat对象：

```
Mat img = imread("test1.jpg");
```
3、此时，因为要对图像进行一些变换，所以我们需要一个新的Mat对象，以存储变换后的图像。这个Mat对象应该拥有下面的性质：

 - 像素值初始化为0 
 - 与原图像有相同的大小和类型
 

```
Mat newImage = Mat::zeros(img.size(), img.type());
```
可以看到到， Mat::zeros 采用 image.size() 和 image.type() 来对Mat对象进行0初始化。

4、现在，为了执行运算 g(i,j) = alpha * f(i,j) + beta ，我们要访问图像的每一个像素。因为是对RGB图像进行运算，每个像素有三个值（R、G、B），所以我们要分别访问它们。下面是访问像素的代码片段：

```
for (int y = 0; y < img.rows; y++)
	{
		for (int x = 0; x < img.cols; x++)
		{
			for (int c = 0; c < 3; c++)
			{
				newImage.at<Vec3b>(y, x)[c] =
			    saturate_cast<uchar>(alpha*(img.at<Vec3b>(y, x)[c]) + beta);
			}
		}
	}
```


最后注意以下两点：

 - 为了访问图像的每一个像素，我们使用这一语法： image.at<Vec3b>(y,x)[c] 其中， y 是像素所在的行， x是像素所在的列， c 是R、G、B（0、1、2）之一。 
 - 因为 alpha * p(i,j) + beta的运算结果可能超出像素取值范围，还可能是非整数（如果 alpha 是浮点数的话），所以我们要用 saturate_cast对结果进行转换，以确保它为有效值。

**saturate_cast原理应当如下：**

```
if(data<0)
	data=0;
else if(data>255)
	data=255;
```
saturate_cast主要是为了防止溢出，对数据进行的一种保护，使赋值的操作合理无错


>PS： 我们可以不用 for 循环来访问每个像素，而是可以直接采用下面这个命令： image.convertTo(new_image, -1,alpha, beta); 
>这里的 convertTo 将执行我们想做的 new_image = a*image + beta 。然而，由于我们想展现访问每一个像素的操作过程，所以选用了for循环的方式。实际上，这两种方式都能返回同样的结果。
