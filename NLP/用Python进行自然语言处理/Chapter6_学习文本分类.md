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
train_set = feature_sets[500:]
test_set = feature_sets[:500]
# 训练贝叶斯分类器
classifier = nltk.NaiveBayesClassifier.train(train_set)
# 分类测试
print classifier.classify(gender_features(u'Neo'))
# 分类器评估
print nltk.classify.accuracy(classifier, test_set)
# 查看哪些特征是有效的
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
------------------------|--------------------------------------
last_letter = u'a'      |     female : male   =     20.6 : 1.0
last_letter = u's'      |       male : female =      8.1 : 1.0
last_letter = u'r'      |       male : female =      4.1 : 1.0
last_letter = u'p'      |       male : female =      4.1 : 1.0

## （2）存储优化
我们有时很不希望feature matrix全部加载到内存，尤其是instance很多，dimension也很多的情况下。我们可以使用nltk.classify.apply_features返回一个不会在内存存储
的对象。
```python
# -*-coding: utf-8 -*-
from nltk.classify import apply_features
train_set = apply_features(gender_features, names[500:])  # 第一个参数为一个函数
test_set = apply_features(gender_features, names[:500])
```
## （3）关于验证集
使用验证集，我们可以生成一个错误列表。观察这个错误列表，我们可能会获得一些对特征的改进。

## （4）文档分类
我们使用影评语料库，将每个评论归类为正面或负面。
```python
# -*-coding: utf-8 -*-
import nltk
import random
from nltk.corpus import movie_reviews


# 获得原始数据集（document：cate对）
documents_cates = [(list(movie_reviews.words(fileid)), category)
                   for category in movie_reviews.categories()
                   for fileid in movie_reviews.fileids(category)]
random.shuffle(documents_cates)
# 将在语料库中频率最高的2000个词作为特征
corpus_words_fre = nltk.FreqDist(w.lower() for w in movie_reviews.words())
words_features = corpus_words_fre.keys()[:2000]
# 文档特征赋值
def document_features(document):
    """
    :param document: list类型
    :return:
    """
    document_words_set = set(document)  # 判断word是否在集合中要比判断word是否在列表中快得多
    features = {}
    for word in words_features:
        features['contains(%s)' % word] = (word in document_words_set)
    return features
# 训练
feature_sets = [(document_features(document), category) for (document, category) in documents_cates]
train_set, test_set = feature_sets[100:], feature_sets[:100]
classifier = nltk.NaiveBayesClassifier.train(train_set)
# 测试准确率
print nltk.classify.accuracy(classifier, test_set)
# 展示最相关的feature
classifier.show_most_informative_features(5)

```
*output*
> 0.69

 Most Informative Features         | .
---------------------------------- |-----------------------------------------
          contains(sans) = True    |         neg : pos    =      9.0 : 1.0
  contains(effortlessly) = True    |         pos : neg    =      7.4 : 1.0
     contains(dismissed) = True    |         pos : neg    =      7.0 : 1.0
    contains(mediocrity) = True    |         neg : pos    =      6.3 : 1.0
        contains(fabric) = True    |         pos : neg    =      6.3 : 1.0

