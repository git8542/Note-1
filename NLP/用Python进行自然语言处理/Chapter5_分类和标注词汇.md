#### 目录

- 1.使用词性标注器
- 2.自动词性标注
	- 默认标注器
	- 正则表达式标注器
	- 查询标注器
- 3.N-gram标注器
	- 划分训练集和测试集
	- N-gram标注
	- 组合标注器
	- 标注生词
	- 存储标注器
	- 性能限制
- 4.基于转换的标注
	- n-gram标注器的缺点
	- Brill标注
- 5.确定词性分类的方向



# 1.使用词性标注器
```python

import nltk
tokens = nltk.word_tokenize(u'I love you, right?', language='english')
print nltk.pos_tag(tokens)

```
output:
> [(u'I', 'PRP'), (u'love', 'VBP'), (u'you', 'PRP'), (u',', ','), (u'right', 'RB'), (u'?', '.')]

# 2.自动词性标注
本书提供了自动词性标注的3个思路。
## （1）默认标注器
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

## （2）正则表达式标注器
基于词尾规则。
比如我们认为以`ing`结尾的词都是动名词
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

## （3）查询标注器
找出前n个最频繁的词，存储他们的标记，然后使用此信息作为查询标注器的模型。
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

# 3.N-gram标注器
## （1）划分训练集和测试集
```python
size = int(len(brown_tagged_sents)*0.9)
train_sents = brown_tagged_sents[:size]
test_sents = brown_tagged_sents[size:]

```
## （2）N-gram标注

**一元标注器**基于一个简单的统计方法：对每个标识符分配这个独特的标识符最有可能的标记。比如对于`frequent`这个单词，它在已标注语料库中既被标注为形容词，也被标注成动词，但是它被标注成形容词的频率高于被标注成动词的，所以我们的一元标注器最终标注另一语料库中的`frequent`时，都将其标注为形容词。

**N元标注器**的上下文是当前标识符和它前面n-1个标识符的词性标记（一元标注器的上下文是整个语料库）。它会给出给定的上下文中最有可能的标记。比如n=2时，语料库中出现`frequent`的上下文有：`n+adj`，`n+v`，`adv+adj`这三种，频率分别为1，2，3.我们的标注器在标注时检测出`frequent`前面的标识符的词性，如果是`n`，则属于上面的前两种情况，这时根据频率，我们将`frequent`在此处判断为`v`；如果`frequent`前面的标识符的词性为`adv`，则认定它在此处的词性应该为`adj`。

使用bigram标注
```python
bigram_tagger = nltk.BigramTagger(tagged_sents)
```

当n增大时，上下文的特异性就会增大，我们要标注的数据中包含训练数据中不存在的上下文的几率也增大

注意：N-gram标注器不应该考虑跨越句子边界的上下文。对于句首的标识符，前面的标记被设置为None（在nltk中）。

## （3）组合标注器

我们可以组合bigram标注器、unigram标注器和默认标注器。

1. 尝试使用bigram标注器标注标识符
2. 如果bigram标注器无法找到一个标记，尝试unigram标注器
3. 如果unigram标注器也无法找到一个标记，使用默认标注器

code：

```python
tagger1 = nltk.DefaultTagger('NN')
tagger2 = nltk.UnigramTagger(train_sents, backoff=tagger1)  # 将已标注语料库划分为训练集和测试集
tagger3 = nltk.BigramTagger(train_sents, backoff=tagger2)  # backoff参数表示回退标注器（未被标注后采取的措施）
tagger3.evaluate(test_sents)
```
（当然我们也可以加上TrigramTagger）
我们也可以忽略那些出现次数少的上下文，比如ntlk.BigramTagger(sents, cutoff=2,backoff=tagger2)将会忽略掉只看到一两次的上下文。

## （4）标注生词
之前对于生词（训练集词汇表）的处理，几乎是全部被默认标注器标注为同一词性。我们可以进一步优化：限制词汇表中只保留最频繁的n个词，即频率较低（比如频率为1）的词就不要出现在词汇表中了，而是将其替换成一个特殊的词`UNK`（写法随意）。这样训练时，unigram标注器可能会学到UNK通常为一个名词，而n-gram标注器也会检测它的其它一些标记中的上下文。真正标注时，如果新词不在词汇表，就将其替换为`UNK`进行标注。

## （5）存储标注器
我们没有必要在每次需要标注时都训练一个标注器，我们可以将一个训练好的标注器保存到文件以后重复使用。
```python
# -*-coding: utf-8 -*-
from cPickle import dump
from cPickle import load

# Save to file
output = open(r'tagger.pkl', 'wb')
dump(tagger, output, -1)
output.close()
# load to program
input = open(r'tagger.pkl', 'rb')
tagger = load(input)
input.close()
```

## （6）性能限制

训练数据中n-gram带来的歧义导致了标注器性能的上界。歧义是指：假设对于bigram，对于frequent这个词来说，训练集中同时出现了n+adj，n+v ，因为它们存在相同的context（前一个
标识符都是名词），我们就称此为歧义。

更多的上下文可以一定程度上消除一些歧义。标记集的大小却影响着上下文，标记集越小，上下文信息就会越少。


# 4.基于转换的标注

## （1）n-gram标注器的缺点
1. n-gram存储表可能过大。尤其是使用了回退标注器的n-gram标注器可能存储trigram和bigram表，这是很大的稀疏阵列，可能数亿条。
2. n-gram从上下文中获得的唯一信息是前面词的标记，而词本身可能是很有用的。

## （2）Brill标注

简介：

Brill标注是一种基于转换的学习。基本思想是：猜每个词的标记，然后返回和修复错误。这样，Brill标注器陆续将一个不良标注的文本转换成一个更好的。不像n-gram标注器，它不计数观测结果，只编制一个转换修正规则链表。还有一点就是：它形成的规则是语言可解释的。

# 5.确定词性分类的方向

回归基本，如何决定一个词属于哪一类？语言学家使用形态学、句法、语义线索确定一个词的词性分类。

（1）**形态学线索**

词的内部结构可能为词分类提供线索。比如以`ness`结尾的词很可能是名词。

（2）**句法线索**

即结合上下文语境。比如出现在名词前的很可能就是形容词。

（3）**语义线索**

一个词的意思对其词汇范畴是一个有用的线索。比如我们知道中文的	`生日`是名词，对应的英文`birthday`也就很可能是名词了。

（4）**新词**

最新被添加到牛津词典词汇大部分都是名词，几乎不可能是介词。
