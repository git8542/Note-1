# <p align="center">Random

1.**关于MLP和CNN的线性问题**

MLP的非线性主要体现在activation function上，CNN的非线性主要体现在卷积，采样操作上，而它的activation function ReLU却是线性的。

2.**关于归一化问题**

树型算法（Decision Tree，RF，GBDT，GBRT等）是不需要归一化的。因为需要归一化方法拟合的是连续或者分段连续函数，而树型算法可以拟合任意间断函数；而且，归一化是为梯度下降这类优化算法服务的，而树型算法的优化算法并不是梯度下降法。

梯度下降法只能对连续或者分段连续函数进行优化，不能对于树型算法这种任意间断函数进行优化。

Boosting Tree Ensemble是在残差下降最快的方向上进行优化，存在梯度下降的概念，但并不是梯度下降法。