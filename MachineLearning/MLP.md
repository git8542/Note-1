目录：

1. 理论
2. 实践
    - 分类
    - 回归
    - Tips
    - warm start

# 1. 理论
## （1）Advantages 
1. Capability to learn non-linear models.
2. Capability to learn models in real-time (on-line learning) (using `partial_fit`在sklearn中)
## （2）Disadvantages
1. MLP with hidden layers have a non-convex loss function where there exists more than one local minimum. Therefore different random weight initializations can lead to different validation accuracy.
2. MLP requires tuning a number of hyperparameters such as the number of hidden neurons, layers, and iterations.
3. MLP is sensitive to feature scaling.

# 2. 实践
## （1）分类
使用[`sklearn.neural_network.MLPClassifier`](http://scikit-learn.org/stable/modules/generated/sklearn.neural_network.MLPClassifier.html#sklearn.neural_network.MLPClassifier)API。

例子：
```python
from sklearn.neural_network import MLPClassifier


X = [[0., 0.], [1., 1.]]
y = [0, 1]
clf = MLPClassifier(solver='lbfgs', alpha=1e-5, hidden_layer_sizes=(5, 2), random_state=1)
clf.fit(X, y)
clf.predict([[2., 2.], [-1., -2.]])

# 打印weights
for matrix in clf.coefs_:
    print(matrix)
    print('\n')
# 打印概率
clf.predict_proba([[2., 2.], [1., 2.]])
```
## （2）回归
使用[`sklearn.neural_network.MLPRegressor()`](http://scikit-learn.org/stable/modules/generated/sklearn.neural_network.MLPRegressor.html#sklearn.neural_network.MLPRegressor)API。

## （3）Tips
1. Since backpropagation has a high time complexity, it is advisable to start with smaller number of hidden neurons and few hidden layers for training.
2. Multi-layer Perceptron is sensitive to feature scaling, so it is highly recommended to scale your data.
3. Finding a reasonable regularization parameter `alpha` is best done using [`GridSearchCV`](http://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html), usually in the range `10.0 ** -np.arange(1, 7)`。
4.  `L-BFGS` converges faster and with better solutions on **small datasets**. For relatively large datasets, however, `Adam` is very robust. It usually converges quickly and gives pretty good performance. `SGD` with momentum or nesterov’s momentum, on the other hand, can perform better than those two algorithms if learning rate is correctly tuned.

## （4）warm_start
If you want more control over stopping criteria or learning rate in `SGD`, or want to do additional monitoring, using `warm_start=True` and `max_iter=1` and iterating yourself can be helpful:
```python
X = [[0., 0.], [1., 1.]]
y = [0, 1]
clf = MLPClassifier(hidden_layer_sizes=(15,), random_state=1, max_iter=1, warm_start=True)
for i in range(10):
    clf.fit(X, y)
    # additional monitoring / inspection 
```

<br />

Ref:

http://scikit-learn.org/stable/modules/neural_networks_supervised.html#neural-network-models-supervised