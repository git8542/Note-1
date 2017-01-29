目录：

1. 理论
2. 实践
    - 寻找最近邻居
    - 分类
        - 算法选择
        - 参数调试
    - 回归
    - 近似最近邻


# 1. 理论

# 2. 实践
## （1）寻找最近邻居

## （2）分类
1. 算法选择
	- 数据分布较为均匀，使用`KNeighborsClassifier`
	- 数据分布不均匀，使用`RadiusNeighborsClassifier`

2. 参数调试
	- `n_neighbors`：
	- `weights`：默认是`uniform`，表示所有neighbor的权重一致；建议使用`distance`，表明按距离给neighbor分配权重。
	- `algorithm`：Algorithm used to compute the nearest neighbors
    		
		- `ball_tree` will use BallTree
		- `kd_tree` will use KDTree
    		- `brute` will use a brute-force search.
    		- `auto` will attempt to decide the most appropriate algorithm based on the values passed to `fit` method.
	
	- `metric`：距离度量，默认是闵可夫斯基距离
	- `p`：闵可夫斯基的范数，默认是`2`，表明2范数，即欧几里得距离
	- `n_jobs`：The number of parallel jobs to run for neighbors search. If `-1`, then the number of jobs is set to the number of CPU cores.

## （3）回归
Neighbors-based regression can be used in cases where the data labels are continuous rather than discrete variables. The label assigned to a query point is computed based the mean of the labels of its nearest neighbors.

回归需要注意的问题和分类基本一致

## （4）近似最近邻
**Approximate nearest neighbor** search methods have been designed to try to speedup query time with high dimensional data.
