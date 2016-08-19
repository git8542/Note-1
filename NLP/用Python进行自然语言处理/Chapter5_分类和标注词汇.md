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
> 
0.130894842572

## 2.正则表达式标注器
基于词尾规则。
比如我们认为以ing结尾的词都是动名词
```python
# -*-coding: utf-8 -*-
import nltk
raw = u'I am playing games!'
tokens = nltk.word_tokenize(raw, language=u'english')
pattern = [
    (r'.*ing$', 'VBG'),  # gerunds
    (r'.*ed$', 'VBD'),  # simple past
    (r'.*es$', 'VBZ'),  # 3rd singular present
    (r'.*ould$', 'MD'),  # modals
    (r'.*\'s$', 'NNS'),  # possessive nouns
    (r'.*s$', 'NNS'),  # plural nouns
    (r'^-?[0-9]+(.[0-9]+)?$', 'CD'),  # cardinal numbers
    (r'.*', 'NN')  # nouns(default)
]
regexp_tagger = nltk.RegexpTagger(pattern)
print regexp_tagger.tag(tokens)

```

## 3.查询标注器
找出100个最频繁的词，存储他们的标记，然后使用此信息作为查询标注器的模型。
```python
# -*-coding: utf-8 -*-
import nltk
from nltk.corpus import brown


fd = nltk.FreqDist(brown.words(categories=u'news'))
cfd = nltk.ConditionalFreqDist(brown.tagged_words(categories=u'news'))
most_freq_words = fd.keys()[:100000]  # 这里size可调，当足够大时，比如这里的十万，模型准确率稳定在93.5%左右
likely_tags = dict((word, cfd[word].max()) for word in most_freq_words)
baseline_tagger = nltk.UnigramTagger(model=likely_tags, backoff=nltk.DefaultTagger('NN'))  # 将其余没标注到的都标注为NN
# 标注
raw = u'I like you very much.'
tokens = nltk.word_tokenize(raw, language=u'english')
print baseline_tagger.tag(tokens)
# 评估
brown_tagged_sents = brown.tagged_sents(categories=u'news')  # 用已标注好的语料库做测试
print baseline_tagger.evaluate(brown_tagged_sents)

```