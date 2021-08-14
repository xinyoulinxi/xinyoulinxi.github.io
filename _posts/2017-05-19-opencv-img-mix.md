---
layout: post
author: "ylvoid"
title:  "OpenCv基础（三）"
subtitle: "图像的混合"
date:  2017-05-19 20:44:24
tags:
    - opencv
    - c++
header-mask: 0.2
header-img: "imgs/OpenCV_logo.png"
---
原理
--

以下解释基于Richard Szeliski所著 Computer Vision: Algorithms and Applications 

**线性混合操作 也是一种典型的二元（两个输入）的 像素操作 ：**
```
g(x) = (1 - alpha)*f_0(x) + alpha *f_1(x)
```
通过在范围 `0 --> 1` 内改变 alpha ，这个操作可以用来对两幅图像或两段视频产生画面叠加 `cross-dissolve`效果。比如在不断递增的过程中，一张图片渐渐被替换为另一张图片（图片减隐的效果）

直接上测试代码吧：

```
void testAddWeight(){
	Mat img = imread("test.jpg");
	Mat result = imread("test_.jpg");
	//test_.jpg和以上的图片尺寸相同
	Mat out;
	//以下的代码将在一段时间内执行减隐效果，达到切换图片时渐隐的效果
	for (double i = 0.0; i < 1; i += 0.01f){
		addWeighted(img, i, result, 1.0 - i, 0.0, out);
		imshow("src1", img);
		imshow("src2", result);
		imshow("outAddWeightImg", out);
		waitKey(30);
	}
	waitKey(0);
}
```
**程序运行结果如下：**

![](/imgs/post/opencv/3-1.png)



具体实现如下
------


1、加载图片
------

为了达到`g(x) = (1 - alpha)*f_0(x) + alpha*f_1(x)`的效果
我们需要两幅输入图像 `(f_{0}(x)` 和 `f_{1}(x))`。

相应地，我们使用常用的方法加载图像：

```
Mat img = imread("test.jpg");
Mat result = imread("test_.jpg");
//test_.jpg和以上的图片尺寸相同
```

2、生成图像 g(x)
-----------

使用函数 addWeighted 可以很方便地实现生成:

```
addWeighted(img, i, result, 1.0 - i, 0.0, out);
```
i代表alpha，`（1.0-i）`代表beta，然后addWeighted()函数将会对图像矩阵做如下运算：

`out = alpha （点乘）img + beta （点乘） result + gamma`

**这里 gamma 对应于我上面代码中被设为 0.0 的参数**

3、创建显示窗口显示图像
----------------------

结束






