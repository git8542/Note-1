#<p align=center> 卷积神经网络

## Overview
Convolutional networks were inspired by biological processes and are variations of **multilayer perceptrons** designed to use **minimal amounts of preprocessing**. They have wide applications in image and video recognition, recommender systems and natural language processing.

## 1. Features
While traditional multilayer perceptron (MLP) models were successfully used for image recognition, due to the full connectivity between nodes they suffer from the curse of dimensionality and thus do not scale well to higher resolution images.

For example, images are only of size 32x32x3 (32 wide, 32 high, 3 color channels), so a single fully connected neuron in a first hidden layer of a regular neural network would have 32*32*3 = 3,072 weights. A 200x200 image, however, would lead to neurons that have 200*200*3 = 120,000 weights.

Such network architecture does not take into account the spatial structure of data, treating input pixels which are far apart and close together on exactly the same footing. Clearly, the full connectivity of neurons is wasteful in the framework of image recognition, and the huge number of parameters quickly leads to overfitting.

Convolutional neural networks are biologically inspired variants of multilayer perceptrons, designed to emulate the behaviour of a visual cortex. These models mitigate the challenges posed by the MLP architecture by exploiting the strong spatially local correlation present in natural images. As opposed to MLPs, CNN have the following **distinguishing features**:

1. **3D volumes of neurons**. The layers of a CNN have neurons arranged in 3 dimensions: width, height and depth. The neurons inside a layer are only connected to a small region of the layer before it, called a receptive field. Distinct types of layers, both locally and completely connected, are stacked to form a CNN architecture.
2. **Local connectivity**: following the concept of receptive fields, CNNs exploit spatially local correlation by enforcing a local connectivity pattern between neurons of adjacent layers. The architecture thus ensures that the learnt "filters" produce the strongest response to a spatially local input pattern. Stacking many such layers leads to non-linear "filters" that become increasingly "global" (i.e. responsive to a larger region of pixel space). This allows the network to first create good representations of small parts of the input, then assemble representations of larger areas from them.
3. **Shared weights**: In CNNs, each filter is replicated across the entire visual field. These replicated units share the same parameterization (weight vector and bias) and form a feature map. This means that all the neurons in a given convolutional layer detect exactly the same feature. Replicating units in this way allows for features to be detected regardless of their position in the visual field, thus constituting the property of translation invariance.

## 2. Blocks

卷积神经网络主要由 卷积层（Convolutional layer）、线性整流层（Rectified Linear Units layer, ReLU layer）、池化层（Pooling layer）、全连接层（ Fully-Connected layer）、损失层（Loss layer）组成。卷积层和池化层可能会在一个模型里重复出现。

一个典型的卷积神经网络拓扑结构如图：
![](http://i.imgur.com/F6jQsOZ.png)
（采样层即池化层）

###2.1 卷积层
结构如图：
![](http://i.imgur.com/dN17hcY.png)

The convolutional layer is the core building block of a CNN. The layer's parameters consist of a set of learnable filters (or kernels), which have a small receptive field, but extend through the full depth of the input volume. During the forward pass, each filter is convolved across the width and height of the input volume, computing the dot product between the entries of the filter and the input and producing a 2-dimensional activation map of that filter.（这里的卷积运算其实是用dot product实现的） As a result, the network learns filters that activate when they see some specific type of feature at some spatial position in the input.Stacking the activation maps for all filters **along the depth** dimension forms the full output volume of the convolution layer.

（每个卷积层是由多个特征图堆积而成，一个过滤器或者称为卷积核产生一个特征图feature map，有时候把特征图称作通道channel；一个特征图其实是一个卷积核提取出来的特征。）

####2.1.1 卷积运算

![](http://i.imgur.com/OQ8M3IK.png)

运算过程：（展示了一个3×3的卷积核在5×5的图像上做卷积的过程）

![](https://raw.githubusercontent.com/stdcoutzyx/Paper_Read/master/blogs/imgs/6.gif)

当stride为1时：输出量的宽度=输入量的宽度 - 卷积核的宽度 + 1

注意：上图体现了卷积层的**局部连接**、**权值共享**特性。

（对于为什么卷积可以用点积实现，暂留疑问？？）

####2.1.2 权值共享
It relies on one reasonable assumption: That if one patch feature is useful to compute at some spatial position, then it should also be useful to compute at a different position. In other words, denoting a single 2-dimensional slice of depth as a depth slice, we constrain the neurons in each depth slice to use the same weights and bias.

####2.1.3 空间排列（Spatial arrangement）

一个输出单元的大小有以下三个量控制：depth, stride 和 zero-padding。


- 深度(depth) : 顾名思义，它控制输出单元的深度，也就是filter的个数，连接同一块区域的神经元个数。又名：depth column
- 步幅(stride)：它控制在同一深度的相邻两个隐含单元，与他们相连接的输入区域的距离。如果步幅很小（比如 stride = 1）的话，相邻隐含单元的输入区域的重叠部分会很多; 步幅很大则重叠区域变少。
- 补零(zero-padding) ： 我们可以通过在输入单元周围补零来改变输入单元整体大小，从而控制输出单元的空间大小。

![](http://i.imgur.com/pBctykS.png)

####2.1.4 非第一卷积层
这里的卷积层的输入是由多个特征图堆叠而成的（刚刚所说的那种卷积层的输入直接是输入层，即输入只有一个特征图），结构见下图：
![](http://i.imgur.com/xifBiTl.png)
上图展示了在四个通道上的卷积操作，有两个卷积核，生成两个通道。其中需要注意的是，四个通道上每个通道对应一个卷积核，先将w2忽略，只看w1，那么在w1的某位置（i,j）处的值，是由四个通道上（i,j）处的卷积结果相加然后再取激活函数值得到的。

###2.2 线性整流层（ReLU layer）
ReLU is the abbreviation of Rectified Linear Units. This is a layer of neurons that applies the non-saturating activation function {\displaystyle f(x)=\max(0,x)} . It increases the nonlinear properties of the decision function and of the overall network without affecting the receptive fields of the convolution layer.

Other functions are also used to increase nonlinearity, for example the saturating hyperbolic **tangent** and the **sigmoid** function. Compared to other functions the usage of ReLU is preferable, because it results in the neural network training several times faster, without making a significant difference to generalisation accuracy.

**Adds**:

The Rectified Linear Unit has become very popular in the last few years. It computes the function f(x)=max(0,x). In other words, the activation is simply thresholded at zero . There are several pros and cons to using the ReLUs:
- (+) It was found to greatly accelerate the convergence of stochastic gradient descent compared to the sigmoid/tanh functions. It is argued that this is due to its linear, non-saturating form.
- (+) Compared to tanh/sigmoid neurons that involve expensive operations (exponentials, etc.), the ReLU can be implemented by simply thresholding a matrix of activations at zero.
- (-) Unfortunately, ReLU units can be fragile during training and can “die”. For example, a large gradient flowing through a ReLU neuron could cause the weights to update in such a way that the neuron will never activate on any datapoint again. If this happens, then the gradient flowing through the unit will forever be zero from that point on. That is, the ReLU units can irreversibly die during training since they can get knocked off the data manifold. For example, you may find that as much as 40% of your network can be “dead” (i.e. neurons that never activate across the entire training dataset) if the learning rate is set too high. With a proper setting of the learning rate this is less frequently an issue.

**Leaky ReLU**

Leaky ReLUs are one attempt to fix the “dying ReLU” problem. Instead of the function being zero when x < 0, a leaky ReLU will instead have a small negative slope (of 0.01, or so). That is, the function computes f(x)=(x<0)(αx)+(x>=0)(x) where α is a small constant.

### 2.3 池化层（Pooling layer）

池化（pool）即下采样（downsamples），目的是为了减少特征图。池化操作对每个深度切片独立，规模一般为 2*2，相对于卷积层进行卷积运算，池化层进行的运算一般有以下几种：

- 最大池化（Max Pooling）。取4个点的最大值。这是最常用的池化方法。
- 均值池化（Mean Pooling）。取4个点的均值。
- 高斯池化。借鉴高斯模糊的方法。不常用。
- 可训练池化。训练函数 ff ，接受4个点为输入，出入1个点。不常用。

最常见的池化层是规模为2*2， stride步幅为2，对输入的每个深度切片进行下采样。每个MAX操作对四个数进行，如下图所示：
![](http://i.imgur.com/NbLPRH3.png)

池化操作将保存深度大小不变。
如果池化层的输入单元大小不是二的整数倍，一般采取边缘补零（zero-padding）的方式补成2的倍数，然后再池化。

### 2.4 全连接层（Fully-connected layer）
Finally, after several convolutional and max pooling layers, the high-level reasoning in the neural network is done via fully connected layers. Neurons in a fully connected layer have full connections to all activations in the previous layer, as seen in regular Neural Networks. Their activations can hence be computed with a matrix multiplication followed by a bias offset.

全连接层及其之后就是一个普通的MLP网络了，它以之前最后一个采样层的输出为输入。可以有多个隐层。

### 2.5 损失层（Loss layer）

The loss layer specifies how the network training penalizes the deviation between the predicted and true labels and is normally the last layer in the network. Various loss functions appropriate for different tasks may be used there. Softmax loss is used for predicting a single class of K mutually exclusive classes. Sigmoid cross-entropy loss is used for predicting K independent probability values in [0,1] . Euclidean loss is used for regressing to real-valued labels [-infty ,+infty ]（负无穷到正无穷） .

softmax loss最常使用，相关介绍见：
[http://ufldl.stanford.edu/wiki/index.php/Softmax%E5%9B%9E%E5%BD%92](http://ufldl.stanford.edu/wiki/index.php/Softmax%E5%9B%9E%E5%BD%92)

## 3. 参数估计：
卷积神经网络的参数估计依旧使用Back Propagation的方法。
详见：
[http://www.moonshile.com/post/juan-ji-shen-jing-wang-luo-quan-mian-jie-xi#toc_15](http://www.moonshile.com/post/juan-ji-shen-jing-wang-luo-quan-mian-jie-xi#toc_15)

## 4. 选择超参数(Choosing hyperparameters)
详见[https://en.wikipedia.org/wiki/Convolutional_neural_network#Choosing_hyperparameters](https://en.wikipedia.org/wiki/Convolutional_neural_network#Choosing_hyperparameters)

## 5. 正则化方法(Regularization methods)
详见[https://en.wikipedia.org/wiki/Convolutional_neural_network#Regularization_methods](https://en.wikipedia.org/wiki/Convolutional_neural_network#Regularization_methods)

## 6. 应用(Applications)
Image recognition、Video analysis、Natural language processing、Drug discovery、Playing Go
详见：[https://en.wikipedia.org/wiki/Convolutional_neural_network#Applications](https://en.wikipedia.org/wiki/Convolutional_neural_network#Applications)

<br>
<br>
References:

1.[https://en.wikipedia.org/wiki/Convolutional_neural_network](https://en.wikipedia.org/wiki/Convolutional_neural_network)

2.[http://www.liuhe.website/index.php?/Articles/single/37](http://www.liuhe.website/index.php?/Articles/single/37)

3.[http://www.moonshile.com/post/juan-ji-shen-jing-wang-luo-quan-mian-jie-xi](http://www.moonshile.com/post/juan-ji-shen-jing-wang-luo-quan-mian-jie-xi)

4.[http://blog.csdn.net/stdcoutzyx/article/details/41596663](http://blog.csdn.net/stdcoutzyx/article/details/41596663)

5.[http://ufldl.stanford.edu/wiki/index.php/Softmax%E5%9B%9E%E5%BD%92](http://ufldl.stanford.edu/wiki/index.php/Softmax%E5%9B%9E%E5%BD%92)

6.[http://deeplearning.net/tutorial/lenet.html](http://deeplearning.net/tutorial/lenet.html)

7.[http://cs231n.github.io/neural-networks-1/](http://cs231n.github.io/neural-networks-1/)
