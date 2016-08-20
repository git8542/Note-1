### 目录



# 1.有监督分类
## （1）性别鉴定（一个分类例子）

```python

# -*-coding: utf-8 -*-
from nltk.corpus import names
import random
import nltk


# 我们将一个人名字的最后一个字母作为特征
def gender_features(name):
    return {u'last_letter': name[-1]}
# 加载原始数据集
name_genders = [(name, 'male') for name in names.words('male.txt')]+(
    [(name, 'female') for name in names.words('female.txt')])
random.shuffle(name_genders)  # 打乱顺序（因为我们要划分测试集和训练集）
# 特征提取
feature_sets = [(gender_features(name), gender) for (name, gender) in name_genders]
# 划分测试集和训练集
train_set = feature_sets[:700]
test_set = feature_sets[700:]
# 训练贝叶斯分类器
classifier = nltk.NaiveBayesClassifier.train(train_set)
# 分类测试
print classifier.classify(gender_features(u'Neo'))
# 分类器评估
print nltk.classify.accuracy(classifier, test_set)
# 查看last_letter这个特征的哪些值是最有效的
classifier.show_most_informative_features(5)

```

*output*
> male
> 
> 0.712865819989
> 
> Most Informative Features
> 
last_letter = u'd'      |       male : female =     28.6 : 1.0
---------------------------------------------------------------
last_letter = u'a'           female : male   =     20.6 : 1.0
last_letter = u's'             male : female =      8.1 : 1.0
last_letter = u'r'             male : female =      4.1 : 1.0
last_letter = u'p'             male : female =      4.1 : 1.0

