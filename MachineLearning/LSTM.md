**目录**

1. LSTM原理
2. 例子
	- 函数预测
	- 序列分类
	- 双向LSTM

# 1. 原理
参考 [Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)

# 2. 例子
例子所使用的工具：python2.7、Keras (后端是tensorflow)

## （1）函数预测
我们预测`f(x) = x * sin(x)`这个函数在未来某段的走势。

*完整代码*：

```python
# -*- coding: utf-8 -*-
# __author__ = 'Brown'
# create 8/20/17 13:40
from __future__ import print_function
from __future__ import unicode_literals

import math

import matplotlib.pyplot as plt
import numpy as np
from keras.layers import LSTM, Dense
from keras.models import Sequential

__author__ = 'Brown'

"""
预测函数：f(x) = x * sin(x)
"""

TIME_STAMPS = 5
HIDDEN_UNITS = 32
BATCH_SIZE = 50
EPOCHS = 200


def generate_data():
    """
    根据函数生成原始数据
    :return: 
    """
    xs = [x * 0.01 for x in range(-2000, 2000, 10)]  # 横坐标
    ys = [x * math.sin(x) for x in xs]  # 纵坐标
    return xs, ys


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
        sequence_data_x.append(ys[i: i + lookback])  # 用前lookback个数据预测下一个数据
        sequence_data_y.append(ys[i + lookback])
    # 将数据集划分为训练集和测试集
    data_size = len(sequence_data_x)
    split_pos = int(data_size * train_ratio)  # 训练集、测试集划分点在原始数据集中位置
    trainX = sequence_data_x[: split_pos]
    trainY = sequence_data_y[:split_pos]
    testX = sequence_data_x[split_pos:]
    testY = sequence_data_y[split_pos:]
    return trainX, trainY, testX, testY


def build_model():
    model = Sequential()
    model.add(LSTM(HIDDEN_UNITS, return_sequences=True, input_shape=(TIME_STAMPS, 1)))
    model.add(LSTM(HIDDEN_UNITS, return_sequences=True))
    model.add(LSTM(HIDDEN_UNITS))
    model.add(Dense(1))
    model.compile(loss='mean_squared_error', optimizer='adam')
    return model


def train_model(model, trainX, trainY):
    model.fit(trainX, trainY, batch_size=BATCH_SIZE, epochs=EPOCHS)


if __name__ == '__main__':
    # 生成数据
    xs, ys = generate_data()
    # 生成序列数据集，并划分训练集和测试集
    trainX, trainY, testX, testY = split_train_test_set(ys, lookback=TIME_STAMPS)
    # 对X进行reshape。原始shape: (batch_size, time_steps),reshape之后的shape: (batch_size, time_steps, 1)
    trainX = np.reshape(trainX, (len(trainX), len(trainX[0]), 1))
    testX = np.reshape(testX, (len(testX), len(testX[0]), 1))
    # 搭建、训练模型
    lstm_model = build_model()
    train_model(lstm_model, trainX, trainY)
    # 对测试集数据进行预测
    predictY = lstm_model.predict(testX)
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

## （2）序列分类
本例子参考自：[http://machinelearningmastery.com/develop-bidirectional-lstm-sequence-classification-python-keras/](http://machinelearningmastery.com/develop-bidirectional-lstm-sequence-classification-python-keras/)


我们的输入序列是一个随机数序列，比如：

	0.63144003 0.29414551 0.91587952 0.95189228 0.32195638 0.60742236 0.83895793 0.18023048 0.84762691 0.29165514

输出序列是一个二元类标序列，比如：

	0 0 0 1 1 1 1 1 1 1
	
输入序列每个元素对应输出序列相同位置的元素。

输出序列类标如何得到呢？在输入序列中，如果当前位置及其以前位置的所有元素累计和大于某个阈值时，当前类标为`1`，否则为`0`。

这里我们设置阈值为序列长度的`1/4`。

*完整代码：*

```python
# -*- coding: utf-8 -*-
# __author__ = 'Brown'
# create 8/17/17 16:19
from __future__ import print_function
from __future__ import unicode_literals

from random import random

from keras.layers import Dense
from keras.layers import LSTM
from keras.layers import TimeDistributed
from keras.models import Sequential
from numpy import array
from numpy import cumsum


def get_sequence(n_timesteps):
    """
    Create a sequence classification instance
    :param n_timesteps: 
    :return: 
    """
    X = array([random() for _ in range(n_timesteps)])
    limit = n_timesteps / 4.0
    y = array([0 if x < limit else 1 for x in cumsum(X)])
    # reshape input and output to be suitable for LSTMs
    X = X.reshape(1, n_timesteps, 1)
    y = y.reshape(1, n_timesteps, 1)
    return X, y


def build_lstm_model():
    model = Sequential()
    model.add(LSTM(20, input_shape=(n_timesteps, 1), return_sequences=True))
    model.add(TimeDistributed(Dense(1, activation='sigmoid')))
    model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['acc'])
    return model


def train_model(model):
    for epoch in range(1000):
        X, y = get_sequence(n_timesteps)
        model.fit(X, y, epochs=1, batch_size=1, verbose=2)


def evaluate(model):
    X, y = get_sequence(n_timesteps)
    yhat = model.predict_classes(X, verbose=0)
    for i in range(n_timesteps):
        print('Expected: {}'.format(y[0, i]), 'Predicted: {}'.format(yhat[0, i]))


if __name__ == '__main__':
    n_timesteps = 10
    model = build_lstm_model()
    train_model(model)
    evaluate(model)

```

## （3）双向LSTM
基本任务和（2）中一模一样，只不过模型有变化。依然参考自 [http://machinelearningmastery.com/develop-bidirectional-lstm-sequence-classification-python-keras/](http://machinelearningmastery.com/develop-bidirectional-lstm-sequence-classification-python-keras/)

*完整代码*

```python
# -*- coding: utf-8 -*-
# __author__ = 'Brown'
# create 8/17/17 16:19
from __future__ import print_function
from __future__ import unicode_literals

from random import random

from keras.layers import Bidirectional
from keras.layers import Dense
from keras.layers import LSTM
from keras.layers import TimeDistributed
from keras.models import Sequential
from numpy import array
from numpy import cumsum


def get_sequence(n_timesteps):
    """
    Create a sequence classification instance
    :param n_timesteps: 
    :return: 
    """
    X = array([random() for _ in range(n_timesteps)])
    limit = n_timesteps / 4.0
    y = array([0 if x < limit else 1 for x in cumsum(X)])
    # reshape input and output to be suitable for LSTMs
    X = X.reshape(1, n_timesteps, 1)
    y = y.reshape(1, n_timesteps, 1)
    return X, y


def build_bilstm_model():
    model = Sequential()
    model.add(Bidirectional(LSTM(20, return_sequences=True), input_shape=(n_timesteps, 1)))
    model.add(TimeDistributed(Dense(1, activation='sigmoid')))
    model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['acc'])
    return model


def train_model(model):
    for epoch in range(1000):
        X, y = get_sequence(n_timesteps)
        model.fit(X, y, epochs=1, batch_size=1, verbose=2)


def evaluate(model):
    X, y = get_sequence(n_timesteps)
    yhat = model.predict_classes(X, verbose=0)
    for i in range(n_timesteps):
        print('Expected: {}'.format(y[0, i]), 'Predicted: {}'.format(yhat[0, i]))


if __name__ == '__main__':
    n_timesteps = 10
    model = build_bilstm_model()
    train_model(model)
    evaluate(model)

```

