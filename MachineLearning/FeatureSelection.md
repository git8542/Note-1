# Feature Selection

### Overview

特征选择是一个重要的数据预处理过程，在现实机器学习任务中，特征提取完毕之后通常先进行特征选择，然后再训练模型。

### Feature Engineering

![](http://7nj1qk.com1.z0.glb.clouddn.com/@/feature_engineering/intro/feature_engineering.jpg)
### Why we do it?

1. Simplification of models to make them easier to interpret by researchers/user.
2. Shorter training times.
3. Enhanced generalization by reducing overfitting(formally, reduction of variance)

目的：

找出最优特征子集

实现：

（1）删除与target无关的特征

（2）删除冗余特征

减少无关特征可以提高计算速度，也可以帮助我们理解特征。

降低特征之间的相关性（删除冗余特征）可以提高模型准确率。

### Approach

- Wrapper - search through the space of subsets, train a model
for current subset, evaluate it on held-out data, iterate...
simple greedy search heuristics:
 - forward selection - start with an empty set, gradually add the
"strongest" features
 - backward selection - start with the full set, gradually remove
the "weakest" features
computationally expensive
- Flter - use N most promissing features according to ranking
resulting from a proxy measure, e.g. from
mutual information
Pearson correlation coefficient
- Embedded methods - feature selection is a part of model
construction

### Extension

[http://dataunion.org/14072.html](http://dataunion.org/14072.html "结合Scikit-learn介绍几种常用的特征选择方法")

### Experience
![a](http://i.imgur.com/EFzPJNS.png)

### QA

>**Q**:特征之间存在较大的相关性会对模型的结果有影响吗？

>**A**:会有。在很多实际的数据当中，往往存在多个互相关联的特征，这时候模型就会变得不稳定，数据中细微的变化就可能导致模型的巨大变化（模型的变化本质上是系数，或者叫参数，可以理解成W），这会让模型的预测变得困难，这种现象也称为多重共线性。例如，假设我们有个数据集，它的真实模型应该是Y=X1+X2，当我们观察的时候，发现Y’=X1+X2+e，e是噪音。如果X1和X2之间存在线性关系，例如X1约等于X2，这个时候由于噪音e的存在，我们学到的模型可能就不是Y=X1+X2了，有可能是Y=2X1，或者Y=-X1+3X2。

<br>
<br>
<br>
<br>

**References**

[https://en.wikipedia.org/wiki/Feature_selection](https://en.wikipedia.org/wiki/Feature_selection)

[http://dataunion.org/20276.html](http://dataunion.org/20276.html)

[http://dataunion.org/14072.html](http://dataunion.org/14072.html)