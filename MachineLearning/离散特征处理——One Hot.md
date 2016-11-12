目录

1. 问题的产生
2. One-Hot编码
3. 在sklearn中使用One-Hot
    - 使用`OneHotEncoder()`
    - 使用`DictVectorizer()`

# 1. 问题的产生
在很多机器学习任务中，特征并不总是连续值，而有可能是**离散值**。

例如，考虑以下的三个特征：
```
["male", "female"]
["from Europe", "from US", "from Asia"]
["uses Firefox", "uses Chrome", "uses Safari", "uses Internet Explorer"]
```
如果将上述特征**用数字表示**，效率会高很多。例如：
```
["male", "from US", "uses Internet Explorer"]    表示为 [0, 1, 3]
["female", "from Asia", "uses Chrome"]    表示为 [1, 2, 1]
```
但是，即使转化为数字表示后，上述数据也不能直接用在我们的分类器中。因为，**分类器往往默认数据是连续的，并且是有序的**。但是，按照我们上述的表示，数字并不是有序的，而是随机分配的。

为了解决这个问题，我们可以使用One-Hot编码。

# 2. One-Hot编码
One-Hot 编码，又称**一位有效编码**，其方法是使用N位状态寄存器来对N个状态进行编码，每个状态都有它独立的寄存器位，并且在任意时候，其中只有一位有效。

例如：
```
对于有6种状态取值的特征，我们有下面的编码方式：

二进制状态码为：000,001,010,011,100,101
One-Hot编码为：000001,000010,000100,001000,010000,100000
```
**也就是说**：对于每一个特征，如果它有`m`个可能值，那么经过One-Hot编码后，就变成了`m`个二元特征。并且，这些特征互斥，每次只有一个激活(为`1`)。

**注意**：
One-Hot编码后可能会出现数据稀疏问题。

# 3. 在sklearn中使用One-Hot

## （1）使用`OneHotEncoder()`

*Code Example：*
```python
In [99]: from sklearn import preprocessing

In [100]: encoder = preprocessing.OneHotEncoder()

In [101]: X = np.array([[0, 0, 3], [1, 1, 0], [0, 2, 1], [1, 0, 2]])

In [102]: encoder.fit_transform(X).toarray()
Out[102]:
array([[ 1.,  0.,  1.,  0.,  0.,  0.,  0.,  0.,  1.],
       [ 0.,  1.,  0.,  1.,  0.,  1.,  0.,  0.,  0.],
       [ 1.,  0.,  0.,  0.,  1.,  0.,  1.,  0.,  0.],
       [ 0.,  1.,  1.,  0.,  0.,  0.,  0.,  1.,  0.]])
```
**解释**：我们编码了3个离散的特征，3个特征的可能取值分别有2，3，4种。所以最终每个instance被映射为9个二元特征。

## （2）使用`DictVectorizer()`
它接收一个dict数组。每个dict代表一个instance，它的key代表特征名称，value代表特征值。

- 如果特征值为数值类型，原样输出；
- 如果特征值为字符串类型，则使用One-Hot编码。

*Code Example：*
```python
In [103]: from sklearn.feature_extraction import DictVectorizer

In [104]: dict_vectorizer = DictVectorizer(sparse=False)

In [105]: x_dict = [{'A': 1, 'B': '1'}, {'A': 2, 'B': '2'}]

In [106]: dict_vectorizer.fit_transform(x_dict)
Out[106]:
array([[ 1.,  1.,  0.],
       [ 2.,  0.,  1.]])
```
**解释**：对于特征`A`，它的数据类型为`int`，向量化之后原样输出；对于特征`B`，它的类型为`str`，表明特征为离散特征，采用One-Hot编码为两个二元特征。

**注意**：dict表示法被视为稀疏矩阵，即它们只包含value非0的特征。也就是说，如果instance A中包含特征a，而instance B中不包含特征a，则说明instance B中特征a的值为0。

<br />

**Ref**:

[one hot 编码及数据归一化](http://blog.csdn.net/dulingtingzi/article/details/51374487)
