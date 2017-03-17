# 1. 相关阅读
[Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)

[A noob’s guide to implementing RNN-LSTM using Tensorflow](http://monik.in/a-noobs-guide-to-implementing-rnn-lstm-using-tensorflow/)

[The Unreasonable Effectiveness of Recurrent Neural Networks](http://karpathy.github.io/2015/05/21/rnn-effectiveness/)

# 2. 例子

这里我们预测`f(x) = x * sin(x)`这个函数。

```python
# -*- coding: utf-8 -*-
import math

import matplotlib.pyplot as plt
import numpy as np
from keras.layers import LSTM, Dense
from keras.models import Sequential

__author__ = 'Brown'

"""
预测函数：f(x) = x * sin(x)
"""


def split_train_test_set(ys, lookback=5, train_ratio=.8):
    """
    :param ys: 原始函数纵坐标
    :param lookback: 每个序列长度
    :param train_ratio: 训练集占比
    :return:
    """
    # 从原始数据生成序列数据集
    sequence_data_x = []
    sequence_data_y = []
    for i in range(len(ys) - lookback):
        sequence_data_x.append(ys[i: i + lookback])
        sequence_data_y.append(ys[i + lookback])
    # 将数据集划分为训练集和测试集
    data_size = len(sequence_data_x)
    split_pos = int(data_size * train_ratio)  # 训练集、测试集划分点在原始数据集中位置
    trainX = sequence_data_x[: split_pos]
    trainY = sequence_data_y[:split_pos]
    testX = sequence_data_x[split_pos:]
    testY = sequence_data_y[split_pos:]
    return trainX, trainY, testX, testY


time_steps = 5
# 生成原始数据
xs = [x * 0.01 for x in range(-2000, 2000, 10)]
ys = [x * math.sin(x) for x in xs]
# 生成序列数据集，并划分训练集和测试集
trainX, trainY, testX, testY = split_train_test_set(ys, lookback=time_steps)
# 对X进行reshape。原始shape: (batch_size, time_steps),reshape之后的shape: (batch_size, time_steps, 1)
trainX = np.reshape(trainX, (len(trainX), len(trainX[0]), 1))
testX = np.reshape(testX, (len(testX), len(testX[0]), 1))
# 搭建模型
model = Sequential()
model.add(LSTM(32, return_sequences=True, input_shape=(time_steps, 1)))
model.add(LSTM(32, return_sequences=True))
model.add(LSTM(32))
model.add(Dense(1))
model.compile(loss='mean_squared_error', optimizer='adam')
# 训练模型
model.fit(trainX, trainY, batch_size=50, epochs=200)
# 对测试集数据进行预测
predictY = model.predict(testX)
# plot测试集中预测结果与实际结果
predict_ys = [x[0] for x in predictY]  # 将二维的预测结果转为一维
actual_ys = testY
test_xs = xs[-len(actual_ys):]
plt.scatter(test_xs, predict_ys, c='r', marker='o')  # 预测结果
plt.scatter(xs, ys, c='b', marker='*')  # 实际结果
plt.show()
```

结果：

![函数拟合](http://i.imgur.com/18lLY6s.png)

测试集上，红色的点表示预测的，蓝色的点表示实际值。

更多信息：[Getting started with the Keras Sequential model](https://keras.io/getting-started/sequential-model-guide/)