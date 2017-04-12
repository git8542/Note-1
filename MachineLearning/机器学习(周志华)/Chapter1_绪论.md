# <p align='center'>第一章 绪论

**本章内容**

1. 引言
2. 基本术语
3. 假设空间
4. 归纳偏好
5. 发展历程
6. 应用现状
7. 附录：领域会议、期刊

### 1. 引言

**Q: 什么是机器学习？**

A: 机器学习致力于研究如何通过计算的手段，利用经验来改善系统自身的性能。这里“经验”通常以“数据”形式存在。机器学习的主要内容是关于在计算机上从“数据”中产生“模型”，即“学习算法”。

### 2. 基本术语

了解以下术语：

- 示例（instance）/样本（sample）
- 属性（attribute）/特征（feature）
- 属性空间（attribute space）/样本空间（sample space）/输入空间（input space）
- 特征向量（feature vector）
- 维度（dimensionality）
- 学习（learning）/训练（training）
- 训练集（training set）
- 假设（hypothesis）
- 学习器（learner）
- 标记（label）
- 样例（example）
- 标记空间（label space）/输出空间
- 分类（classification）
- 回归（regression）
- 测试（testing）、测试样本（testing sample）
- 聚类（clustering）
- 监督学习（supervised learning）、无监督学习（unsupervised learning）
- 泛化能力（generalization）
- 分布（distribution）、独立同分布（independent and identically distributed，简写 i.i.d.）

对于分类：

- 二分类（binary classification）
- 正类（positive class）
- 反类（negative class）
- 多分类（multi-class classification）

**注意**：

**了解sample和example的区别**

拥有了标记信息的示例称为样例

### 3. 假设空间

1. “从样例中学习”是一个归纳过程，也称“归纳学习”（inductive learning）
2. “归纳学习”不是“机械学习”。
3. 对于归纳学习，广义上指“从样例中学习”，狭义指“概念学习”。最基本的概念学习是布尔概念学习。
4. 我们可以把学习过程看作一个在所有假设组成的空间中进行搜索的过程，搜索目标是找到与训练集匹配的假设。可以有很多策略对假设空间进行搜索，例如自顶向下、从一般到特殊，或是自底向上、从特殊到一般，搜索过程中可以不断删除与正例不一致的假设、和（或）与反例一致的假设。
5. 存在一个与训练集一致的“假设集合”，我们称之为“版本空间”（version space）。

### 4. 归纳偏好

1. 一个模型对应了假设空间中的一个假设。
2. 版本空间中的假设在对于一个新样本做出预测时，可能会出现不一致的情况。但对于一个具体算法而言，必须产生一个模型，即选择一个假设。这种选择基于某种偏好（bias）进行。
3. 算法在学习过程中对某种类型假设的偏好，称为“归纳偏好”（inductive bias），简称偏好。
4. 引导算法建立正确偏好的一般性原则 —— **奥卡姆剃刀（Occam's razor）**：“若有多个假设与观察一致，则选最简单的那个。” 奥卡姆剃刀并非科学研究中唯一可行的假设选择原则，伊壁鸠鲁提出的“多释原则”（principle of multiple explanations）主张保留与经验观察一致的所有假设，这与集成学习方面的研究更吻合。
5. 奥卡姆剃刀原则并不保证选择绝对合理，**没有免费午餐定理**（**NFL定理**）表明：当所有“问题”出现的机会相同、或者说所有问题同等重要时，所有学习算法的期望性能都相同。
6. **NFL定理**最重要的寓意是让我们清楚地认识到，脱离具体问题，空泛地谈论“什么学习算法更好”毫无意义，因为若考虑所有潜在的问题，则所有学习算法都一样好。

### 5. 发展历程

**1.机器学习的分类**

E.A.Feigenbaum等人将机器学习划分为“机械学习”、“示教学习”、“类比学习”、“归纳学习”。
“机械学习”即“死记硬背式学习”；“示教学习”和“类比学习”类似于“从指令中学习”和“通过观察和发现学习”；“归纳学习”广义上是“从样例中学习”，它涵盖了监督学习、非监督学习，我们所讲的机器学习大部分都属于“归纳学习”。

**2.“从样例学习”发展历程**

- 二十世纪八十年代：  **符号主义学习**
- 二十世纪九十年代中期之前：  **基于神经网络的连接主义学习**
- 二十世纪九十年代中期：  **统计学习**
- 二十一世纪初：  **连接主义学习**卷土重来，掀起“深度学习”热潮

### 6. 应用现状

- 在**计算机科学的分支领域**中，多媒体技术、图形学、网络通讯、软件工程、芯片设计、计算机视觉、自然语言处理等领域都有用到。
- 在某些**交叉学科**中，比如生物信息学。

### 7. 领域会议、期刊
**机器学习领域**：

会议：ICML，NIPS，COLT，ECML，ACML

期刊：Journal of Machine Learning Research，Machine Learning

国内会议： 中国机器学习大会CCML、机器学习及其应用研讨会MLA

**人工智能领域：**

会议：IJCAI，AAAI 

期刊：Artificial Intelligence，Journal of Artificial Intelligence Research

**数据挖掘领域**：

会议：KDD，ICDM

期刊：ACM Transactions on Knowledge Discovery from Data、Data Mining and Knowledge Discovery

**计算机视觉与模式识别**：

会议：CVPR

期刊：IEEE Transactions on Pattern Analysis and Machine Intelligence

**神经网络领域**：

期刊： Neural Computation、IEEE Transactions on Neural Networks and Learning Systems

**统计学领域**：

期刊： Annals of Statistics
