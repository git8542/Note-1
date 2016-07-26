#Normalization

###简介

**normalization**这里翻译为**归一化**。

###1.为什么要进行归一化

**1）可以加速（梯度下降法）的收敛速度**

如下图所示，蓝色的圈圈图代表的是两个特征的等高线。其中左图两个特征X1和X2的区间相差非常大，X1区间是[0,2000]，X2区间是[1,5]，其所形成的等高线非常尖。当使用梯度下降法寻求最优解时，很有可能走“之字型”路线（垂直等高线走），从而导致需要迭代很多次才能收敛；

而右图对两个原始特征进行了归一化，其对应的等高线显得很圆，在梯度下降进行求解时能较快的收敛。

因此如果机器学习模型使用梯度下降法求最优解时，归一化往往非常有必要，否则很难收敛甚至不能收敛

![](http://images.cnitblog.com/blog2015/522490/201504/192105553858119.png)


**2）对于某些算法，不归一化得到的结果是有问题的**

> Since the range of values of raw data varies widely, in some machine learning algorithms, objective functions will not work properly without normalization. For example, the majority of classifiers calculate the distance between two points by the Euclidean distance. If one of the features has a broad range of values, the distance will be governed by this particular feature. Therefore, the range of all features should be normalized so that each feature contributes approximately proportionately to the final distance.

### 2.如何归一化


**1）线性归一化**

![线性归一化](https://upload.wikimedia.org/math/7/6/5/76512b142c1b7e27e8a7e7eb1fc11225.png)

这种方法将特征范围归一到[0,1]

**2）标准差归一化（标准化）**

经过处理的数据符合标准正态分布，即均值为0，标准差为1，其转化函数为：

![](http://webdataanalysis.net/wp-content/uploads/2010/02/z-score.png)

其中 μ 为所有样本数据的均值，σ 为所有样本数据的标准差。

**3）非线性归一化**

经常用在数据分化比较大的场景（有些数值很大，有些很小）。通过一些数学函数，将原始值进行映射。该方法包括 log、指数，正切等。需要根据数据分布的情况，决定非线性函数的曲线，比如log(V, 2)还是log(V, 10)等。

**4）范数归一化**

> Another option that is widely used in machine-learning is to scale the components(分量) of a feature vector such that the complete vector has length one. This usually means dividing each component by the Euclidean length of the vector. In some applications (e.g. Histogram features) it can be more practical to use the L1 norm (i.e. Manhattan Distance, City-Block Length or Taxicab Geometry) of the feature vector:

![](https://wikimedia.org/api/rest_v1/media/math/render/svg/de841601758441c9441bd73f0025264e181cec58)

> This is especially important if in the following learning steps the Scalar Metric is used as a distance measure.


### 3.归一化，标准化，特征缩放的关系

这三个名词初学者很容易搞混。对数据进行标准差归一化就是标准化，可见标准化是归一化的一种情况。无论归一化还是标准化，都属于特征缩放（feature scaling）。

<br>
<br>
####ref:

[https://en.wikipedia.org/wiki/Feature_scaling](https://en.wikipedia.org/wiki/Feature_scaling) 

[http://www.cnblogs.com/LBSer/p/4440590.html](http://www.cnblogs.com/LBSer/p/4440590.html)

[http://blog.sina.com.cn/s/blog_66a6172c0102v3em.html](http://blog.sina.com.cn/s/blog_66a6172c0102v3em.html)

[https://www.quora.com/What-is-the-difference-between-normalization-standardization-and-regularization-for-data](https://www.quora.com/What-is-the-difference-between-normalization-standardization-and-regularization-for-data)