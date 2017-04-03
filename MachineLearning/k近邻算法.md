目录：

1. 理论
    - 基本原理
2. 实践
    - 寻找最近邻居
    - 分类
        - 算法选择
        - 参数调试
    - 回归
    - 近似最近邻


# 1. 理论
## （1）基本原理
给定一个训练样本集，对于新的输入样本，在训练样本中找到与该样本距离最近的k个样本，取这k个样本的多数类别作为新样本的类别。

KNN的三个基本要素：**k-值选择**、**距离度量**、**分类决策规则**。**KNN在决策边界非常不规则时表现很好**。
# 2. 实践
## （1）寻找最近邻居
你可能有这种需求：从很多数据点中找出与某一点最相近的k个点。这时可以使用sklearn提供的接口[`sklearn.neighbors.NearestNeighbors`](http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.NearestNeighbors.html#sklearn.neighbors.NearestNeighbors)

*NearestNeighbors使用Example*：
```python
>>> from sklearn.neighbors import NearestNeighbors
>>> import numpy as np
>>> X = np.array([[-1, -1], [-2, -1], [-3, -2], [1, 1], [2, 1], [3, 2]])
>>> nbrs = NearestNeighbors(n_neighbors=2, algorithm='ball_tree').fit(X)
>>> distances, indices = nbrs.kneighbors(X)
>>> indices                                           
array([[0, 1],
       [1, 0],
       [2, 1],
       [3, 4],
       [4, 3],
       [5, 4]]...)
>>> distances
array([[ 0.        ,  1.        ],
       [ 0.        ,  1.        ],
       [ 0.        ,  1.41421356],
       [ 0.        ,  1.        ],
       [ 0.        ,  1.        ],
       [ 0.        ,  1.41421356]])
```

[NearestNeighbors](http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.NearestNeighbors.html#sklearn.neighbors.NearestNeighbors)是对[KDTree](http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.KDTree.html#sklearn.neighbors.KDTree)和[BallTree](http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.BallTree.html#sklearn.neighbors.BallTree)的封装，所以你可以直接使用KDTree和BallTree接口。

*KDTree使用Example*：
```python
>>> from sklearn.neighbors import KDTree
>>> import numpy as np
>>> X = np.array([[-1, -1], [-2, -1], [-3, -2], [1, 1], [2, 1], [3, 2]])
>>> kdt = KDTree(X, leaf_size=30, metric='euclidean')
>>> kdt.query(X, k=2, return_distance=False)          
array([[0, 1],
       [1, 0],
       [2, 1],
       [3, 4],
       [4, 3],
       [5, 4]]...)
```
## （2）分类
1. 算法选择
	- 数据分布较为均匀，使用[`KNeighborsClassifier`](http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.KNeighborsClassifier.html#sklearn.neighbors.KNeighborsClassifier)
	- 数据分布不均匀，使用[`RadiusNeighborsClassifier`](http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.RadiusNeighborsClassifier.html#sklearn.neighbors.RadiusNeighborsClassifier)。特征维度较高时，这种方法作用不大。

2. 参数调试
	- `n_neighbors`：
	- `weights`：默认是`uniform`，表示所有neighbor的权重一致；建议使用`distance`，表明按距离给neighbor分配权重。
	- `algorithm`：Algorithm used to compute the nearest neighbors
    		
		- `ball_tree` will use BallTree
		- `kd_tree` will use KDTree
    		- `brute` will use a brute-force search.
    		- `auto` will attempt to decide the most appropriate algorithm based on the values passed to `fit` method.
	- `leaf_size`：当树中叶节点少于此值时，查询转化为暴力搜索。
	- `metric`：距离度量，默认是闵可夫斯基距离
	- `p`：闵可夫斯基的范数，默认是`2`，表明2范数，即欧几里得距离
	- `n_jobs`：The number of parallel jobs to run for neighbors search. If `-1`, then the number of jobs is set to the number of CPU cores.

## （3）回归
Neighbors-based regression can be used in cases where the data labels are continuous rather than discrete variables. The label assigned to a query point is computed based the mean of the labels of its nearest neighbors.

回归需要注意的问题和分类基本一致

## （4）近似最近邻
**Approximate nearest neighbor** search methods have been designed to try to speedup query time with high dimensional data.

[More Info——> Approximate Nearest Neighbors](http://scikit-learn.org/stable/modules/neighbors.html#approximate-nearest-neighbors)

<br />

**Ref**

http://scikit-learn.org/stable/modules/neighbors.html
