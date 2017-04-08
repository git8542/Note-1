目录：
1. 基本原理
2. 实践

# 1. 基本原理
LDA全称Latent Dirichlet Allocation，译为**隐含狄利克雷分布**，是一种主题模型。它可以给出每篇文档的主题概率分布，也可以给出每个主题的单词概率分布。

它基于以下假设：一篇文章由多个主题组成，文章中的每一个词都是由其中的某个主题生成。

然后这些概率分布如何获得呢？通过**吉布斯采样**估计。

<br />

[More Info——>隐含狄利克雷分布](https://zh.wikipedia.org/wiki/%E9%9A%90%E5%90%AB%E7%8B%84%E5%88%A9%E5%85%8B%E9%9B%B7%E5%88%86%E5%B8%83)

[More Info——>What is a good explanation of Latent Dirichlet Allocation?](https://www.quora.com/What-is-a-good-explanation-of-Latent-Dirichlet-Allocation/answer/Edwin-Chen-1?srid=xVbk)

# 2. 简单使用
*Code Example*：
```python
# -*- coding: utf-8 -*-
from gensim import corpora
from gensim import models

__author__ = 'Brown'

texts = [
    'The result of our cleaning stage is texts is'.split(),
    'Synonyms for nice at Thesaurus.com with free online thesaurus'.split()
]
# 建立一个word:integer_id的映射
dictionary = corpora.Dictionary(texts)
# 将每篇文章用(doc_id, freq)的list表示
corpus = [dictionary.doc2bow(text) for text in texts]
# 建立LDA模型，选择topic个数为2
model = models.ldamodel.LdaModel(corpus, num_topics=2, id2word=dictionary, passes=20)
# 打印每个topic下的5个单词
for topic_idx, topic_words in model.show_topics(num_topics=2, num_words=5):
    print(topic_words)
# 打印每个文档对应的topic
for topics in model.get_document_topics(corpus):
    print(topics)
```
*Output*:
```
0.086*"Synonyms" + 0.086*"with" + 0.086*"at" + 0.086*"thesaurus" + 0.086*"free"
0.143*"is" + 0.086*"The" + 0.086*"result" + 0.086*"our" + 0.086*"texts"

[(0, 0.051879848392141083), (1, 0.94812015160785901)]
[(0, 0.94786797319471705), (1, 0.052132026805282929)]
```
<br />

**Ref**:

https://rstudio-pubs-static.s3.amazonaws.com/79360_850b2a69980c4488b1db95987a24867a.html
