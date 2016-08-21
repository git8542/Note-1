### 目录
- 1.有监督分类
    - 性别鉴定（一个分类例子）
    - 存储优化
    - 关于验证集
    - 文档分类
    - 再谈词性标注
    - 序列分类
- 2.有监督分类的更多实例
    - 识别对话行为类型
    - 识别文字蕴含 RTE
- 3.分类（略）
- 4.关于模型的阐述
    - 模型可以告诉我们什么



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

## （5）再谈词性标注
了解了分类，我们可以把词性标注转化为**分类**问题来做。

**特征的选取**可以考虑：词后缀、词长、上下文信息（比如前一个词）等。

## （6）序列分类
相关的分类任务之间可能存在依赖关系。序列分类有很多种，比较普通的一种是**连续序列分类**
，或者称为**贪婪序列分类**。比如在Chapter5的词性标注例子中，我们使用了
bigram标注器，它一开始为句子的第一个词选择标记，然后为每个随后的词选择标记，基
于词本身和前面词的预测结果。

*使用序列分类器进行词性标注*
```python
# -*-coding: utf-8 -*-
import nltk
from nltk.corpus import brown


# 特征提取函数
def pos_features(sentence, i, history):
    """
    :param sentence: 当前句子列表
    :param i: 当前标识符索引
    :param history: 此句子中，当前标识符之前所有标识符的tag（词性）列表
    :return:
    """
    features = {
        'suffix(1)': sentence[i][-1:],
        'suffix(2)': sentence[i][-2:],
        'suffix(3)': sentence[i][-3:]
    }
    if i == 0:
        features['prev-word'] = '<START>'
        features['prev-tag'] = '<START>'
    else:
        features['prev-word'] = sentence[i-1]
        features['prev-tag'] = history[i-1]
    return features


# 建立序列分类器
class ConsecutivePosTagger(nltk.TaggerI):
    def __init__(self, train_sents):
        train_set = []
        for tagged_sent in train_sents:
            untagged_sent = nltk.tag.untag(tagged_sent)
            history = []
            for i, (word, tag) in enumerate(tagged_sent):
                featureset = pos_features(untagged_sent, i, history)
                train_set.append((featureset,  tag))
                history.append(tag)
        self.classifier = nltk.NaiveBayesClassifier.train(train_set)

    def tag(self, sentence):
        history = []
        for i, word in enumerate(sentence):
            featureset = pos_features(sentence, i, history)
            tag = self.classifier.classify(featureset)
            history.append(tag)
        return zip(sentence, history)
# 测试分类器
tagged_sents = brown.tagged_sents(categories='news')
size = int(len(tagged_sents)*0.1)
train_sents, test_sents = tagged_sents[size:], tagged_sents[:size]
tagger = ConsecutivePosTagger(train_sents)
print tagger.evaluate(test_sents)

```
*output*
> 0.798052851182

**连续序列分类弊处**：

一旦作出决定，无法更改。

**其它常见的序列分类模型**：转型联合分类，序列打分

转型联合分类工作原理：为输入的标签创建一个初始值，然后反复提炼那个值，尝试修复相关
输入之间的不一致。比如上节的Brill标注器。

序列打分为词性标记所有可能的序列打分，选择总分最高的序列。比如隐马尔科夫模型。

# 2.有监督分类的更多实例

## （1）识别对话行为类型

对话行为类型可能是：问候、问题、回答、断言、说明等。我们可以基于nltk包中NPS聊天语料库
来进行对话行为类型识别。

```python
# -*-coding: utf-8 -*-
import nltk
from nltk.corpus import nps_chat


# 获取语料库
posts = nps_chat.xml_posts()[:10000]


# 定义特征提取函数(特征是此语料库中所有单词)
def dialogue_act_features(post):
    features = {}
    for word in nltk.word_tokenize(post):
        features['contains(%s)' % word.lower()] = True  # 单词出现在此post中时，此特征值为True
    return features
# 构造特征数据集
featuresets = [
    (dialogue_act_features(post.text), post.get('class')) for post in posts
]
# 其实这里的所有instance维度并不相同（可能是作者失误的地方）
# 或者将其添加到分类器后，分类器会处理维度不同这种情况（将语料库所有标识符视为特征）

# 划分测试集和训练集
size = int(len(featuresets)*0.1)
train_set, test_set = featuresets[size:], featuresets[:size]
# 训练
classifier = nltk.NaiveBayesClassifier.train(train_set)
# 评估
print nltk.classify.accuracy(classifier, test_set)

```

## （2）识别文字蕴含 RTE
RTE是指：根据一个text作为context，判断hypothesis的正确性。

这里只做很简单的rte，我们将问题转化为分类问题，给定一个text/hypothesis对，预测其类标
（true or false）。
```python
# 此函数从text/hypothesis对中提取特征
def rte_features(rtepair):
    extractor = nltk.RTEFeatureExtractor(rtepair)
    features = {}
    features['word_overlap'] = len(extractor.overlap('word'))  # text/hypothesis单词的重叠程度
    features['word_hyp_extra'] = len(extractor.hyp_extra('word'))  # hypothesis中有而text中没有的词的数量
    features['ne_overlap'] = len(extractor.overlap('ne'))  # text/hypothesis中命名实体的重叠程度
    features['ne_hyp_extra'] = len(extractor.hyp_extra('ne'))  # hypothesis中有而text中没有的实体的数量
    return features
```
（这个分类器效果并不好，更多的研究超过本书范围）

# 3.分类技术（略）

# 4.关于模型的阐述

## 模型可以告诉我们什么
处理语言模型时一个重要的思考是**描述型模型**和**解释型模型**之间的区别。

描述型模型捕获数据中的模式，但并不提供任何有关数据包含这些模式的原因的信息。
解释型模型试图捕获造成语言模式的属性和关系。大多数从语料库中构建的模型是描述型
模型，它可以告诉我们哪个特征和一个给定的模式或结构有关，但如何相关，为何相关却不在它的
范畴之内。
