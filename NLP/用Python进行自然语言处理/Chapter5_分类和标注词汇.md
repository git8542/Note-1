# 1.使用词性标注器
```python

import nltk
tokens = nltk.word_tokenize(u'I love you, right?', language='english')
print nltk.pos_tag(tokens)

```
output:
> [(u'I', 'PRP'), (u'love', 'VBP'), (u'you', 'PRP'), (u',', ','), (u'right', 'RB'), (u'?', '.')]

# 2.自动词性标注
本书提供了自动词性标注的四个思路。
## 1.默认标注器
默认标注器是将所有的标识符分配相同的标记。使用最有可能的标记标注每个词。

（可利用已标注的语料库进行第一步分析）

Step：

（1）获取最有可能的标注

（2）给所有的标识符标注

e.g.：

```python
# -*-coding: utf-8 -*-
import nltk
from nltk.corpus import brown
# 获取最可能的tag
tags = [tag for (word, tag) in brown.tagged_words(categories=u'news')]
fre_tag = nltk.FreqDist(tags).max()
# 标注
raw = u'I like you very much.'
tokens = nltk.word_tokenize(raw, language=u'english')
default_tagger = nltk.DefaultTagger(fre_tag)
print default_tagger.tag(tokens)
# 评估
brown_tagged_sents = brown.tagged_sents(categories=u'news')  # 用已标注好的语料库做测试
print default_tagger.evaluate(brown_tagged_sents)


```
输出：
> [(u'I', u'NN'), (u'like', u'NN'), (u'you', u'NN'), (u'very', u'NN'), (u'much', u'NN'), (u'.', u'NN')]
0.130894842572

## 2.正则表达式标注器
基于词尾规则。