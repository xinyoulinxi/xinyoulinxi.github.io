---
layout: post
title:  "SIFT算法的特征点筛选和DMatch、Keypoint描述"
date:    2018-01-08 00:29:25
tags:
    - 关键点检测和追踪
    - AR
    - CV
    - 技术细节
author: "ylvoid"
subtitle: ""
catalog: false
header-style: text
---

# SIFT算法的特征点筛选和DMatch、Keypoint描述
## SIFT算法描述
`SIFT（Scale-invariant feature transform）`是一种检测局部特征的算法，该算法通过求一幅图中的特征点（interest points,or corner points）及其有关`scale` 和 `orientation` 的描述子得到特征并进行图像特征点匹配
这个算法具有比较良好的尺度不变性和旋转不变形
## KeyPoint
`KeyPoint`类的成员变量和描述
```
class KeyPoint
{
Point2f  pt;     //特征点坐标
float  size;     //特征点邻域直径
float  angle;    //特征点的方向，值为0~360，负值表示不使用
float  response; //特征点的响应强度，代表了该点是特征点的稳健度，可以用于后续处理中特征点排序
int  octave;     //特征点所在的图像金字塔的层组
int  class_id;   //用于聚类的id
}
```
特征点主要保存了特征点的位置、领域直径、方向、响应强度、金字塔的层组、聚类ID
# DMatch
`DMatch`，相信用过`FAST、SURF、ORB`等特征点匹配和识别算法的都遇到过这个类型，这个类型最主要的就是存储了两个特征点在各自`keyPoints`中的下标值，然后通过`drawMatches`的API实现两张图片的特征点连线。


对于如下的代码：
```
Mat input1 = imread("img2.jpg", 0);
Mat input2 = imread("img1.jpg", 0);
SiftFeatureDetector detector;
vector<KeyPoint> keypoints1, keypoints2;
detector.detect(input1, keypoints1);
detector.detect(input2, keypoints2);

SiftDescriptorExtractor extractor;
Mat descriptor1, descriptor2;

BruteForceMatcher<L2<float>> matcher;
vector<DMatch> matches;

extractor.compute(input1, keypoints1, descriptor1);
extractor.compute(input2, keypoints2, descriptor2);
matcher.match(descriptor1, descriptor2, matches);

```
我们获得了两张图的特征点，然后通过`BruteForceMatcher<L2<float>>::match`函数获得了`matches`

这个`matchs`里面存储的就是相对于`keypoints1`和`keypoints2`这两个特征点组对应点的下标值和其欧拉距离

其实现如下：
```
struct CV_EXPORTS DMatch
{
    DMatch() : queryIdx(-1), trainIdx(-1), imgIdx(-1), distance(std::numeric_limits<float>::max()) {}
    DMatch( int _queryIdx, int _trainIdx, float _distance ) :
            queryIdx(_queryIdx), trainIdx(_trainIdx), imgIdx(-1), distance(_distance) {}
    DMatch( int _queryIdx, int _trainIdx, int _imgIdx, float _distance ) :
            queryIdx(_queryIdx), trainIdx(_trainIdx), imgIdx(_imgIdx), distance(_distance) {}

    int queryIdx; // query descriptor index
    int trainIdx; // train descriptor index
    int imgIdx;   // train image index
ss
    float distance;

    // less is better
    bool operator<( const DMatch &m ) const
    {
        return distance < m.distance;
    }
};
```
成员变量意义如下：（针对如上的示例代码）

-   `queryIdx `  ：特征点在`keypoints1`中的下标号
-   `trainIdx `  ：特征点在`keypoints2`中的下标号
-   `imgIdx`     ：当前匹配点对应训练图像（如果有若干个）的索引，
如果只有一个训练图像跟查询图像配对，即两两配对，则`imgIdx`=0
- `distance`：对应特征点之间的欧氏距离，越小表明匹配度越高

通过其`operator < `重载函数可以看出，`distance`越小，代表`DMatch`的匹配率越高，同时代表`DMatch`中的两个对应特征点匹配度越高

所以，基于这个原理，我们可以得出一个通过`SIFT`算法(任意方法得到的`KeyPoints`都可以)得到的`KeyPoints`进行优化筛选的方案 —— 通过比较`DMatch`的欧式距离的方式去筛选出欧式距离较小的`DMatch`
这样筛选出来的的`DMatch`就代表了更好的匹配性，我们可以为此做一个测试

## 不进行欧拉距离比对筛选的特征点匹配
这里我们直接使用`SIFT`算法找出`keyPoints`然后直接进行比对

**代码如下** ：
```
int main()
{
	Mat input1 = imread("img2.png", 1);
	Mat input2 = imread("img1.png", 1);
	Mat imgGray1, imgGray2;
	//转换灰度图
	cvtColor(input1, imgGray1, CV_BGR2GRAY);
	cvtColor(input2, imgGray2, CV_BGR2GRAY);
	SiftFeatureDetector detector;
	vector<KeyPoint> keypoint1, keypoint2;
	detector.detect(imgGray1, keypoint1);
	detector.detect(imgGray2, keypoint2);

	SiftDescriptorExtractor extractor;
	Mat descriptor1, descriptor2;

	BruteForceMatcher<L2<float>> matcher;
	vector<DMatch> matches;

	extractor.compute(imgGray1, keypoint1, descriptor1);
	extractor.compute(imgGray2, keypoint2, descriptor2);
	
	matcher.match(descriptor1, descriptor2, matches);
	Mat img_matches;
	drawMatches(input1, keypoint1, input2, keypoint2, matches, img_matches, DrawMatchesFlags::NOT_DRAW_SINGLE_POINTS);
	imshow("SIFT_Match_Image", img_matches);
	waitKey();
	return 0;
}
```
**效果如图**：
![](/imgs/post/track-1.png)

可以发现，这个效果是非常的差的，所有的能够识别到的特征点都被进行了匹配，这样势必导致了大量的错配点和重复点的情况，这并不是我们想要的特征点比对。我们要得是很精准的，少量的特征点

这时候我们就需要用上面我们想到的利用欧拉距离的比对进行筛选的当时选出最优的几个特征点进行匹配
## 使用欧拉距离进行筛选的匹配
**代码如下：**
```

int main()
{
	Mat input1 = imread("img2.png", 1);
	Mat input2 = imread("img1.png", 1);
	Mat imgGray1, imgGray2;
	//转换灰度图
	cvtColor(input1, imgGray1, CV_BGR2GRAY);
	cvtColor(input2, imgGray2, CV_BGR2GRAY);
	SiftFeatureDetector detector;
	vector<KeyPoint> keypoint1, keypoint2;
	detector.detect(imgGray1, keypoint1);
	detector.detect(imgGray2, keypoint2);

	SiftDescriptorExtractor extractor;
	Mat descriptor1, descriptor2;

	BruteForceMatcher<L2<float>> matcher;
	vector<DMatch> matches;

	extractor.compute(imgGray1, keypoint1, descriptor1);
	extractor.compute(imgGray2, keypoint2, descriptor2);
	
	matcher.match(descriptor1, descriptor2, matches);
	//特征点排序
	sort(matches.begin(), matches.end());
	//获取排名前10个的匹配度高的匹配点集
	vector<KeyPoint> goodImagePoints1, goodImagePoints2;

	vector<DMatch> matchesVoted;

	for (int i = 0; i<10; i++)
	{
		DMatch dmatch;
		dmatch.queryIdx = i;
		dmatch.trainIdx = i;

		matchesVoted.push_back(dmatch);
		goodImagePoints1.push_back(keypoint1[matches[i].queryIdx]);
		goodImagePoints2.push_back(keypoint2[matches[i].trainIdx]);
	}

	Mat img_matches;
	std::vector< DMatch > emptyVec;
	drawMatches(input1, goodImagePoints1, input2, goodImagePoints2, matchesVoted, img_matches, DrawMatchesFlags::NOT_DRAW_SINGLE_POINTS);
	imshow("SIFT_Match_Image", img_matches);
	waitKey();
	return 0;
}
```
我们这次并不是仅仅找出了所有的`keyPoints`，还根据匹配得到的`dmatch`数组，进行欧拉距离的排序，然后选出排名前n的数据，这样得到的匹配项，就是我们想要的比较好的特征点匹配

**效果如图：**

![](/imgs/post/track-2.png)
可以看到，效果非常的不错

想一想，通过读`DMatch`和`keyPoint`的源码，我们就可以知道如何优化和筛选`SIFT`算法得到的大量`keyPoint`，并达到非常好的匹配效果
所以编程不能总是只顾着埋头实现功能，自己真正的竞争力还是要自己去看原理得到
