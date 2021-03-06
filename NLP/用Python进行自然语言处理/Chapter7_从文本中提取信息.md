### 目录

- 1.信息提取
    - 信息提取定义
    - 信息提取流程
- 2.分块（chunking）
    - 动名词分块（牛刀小试）
    - 用正则表达式chunk
    - chink（加缝隙）
    - chunk的表示：tag表示和tree表示
- 3.开发和评估分块器
    - 读取IOB格式
    - 分块语料库
    - 简单评估分块器
    - 使用unigram标注器建立分块器
    - 训练基于分类器的分块器
- 4.命名实体识别（NER）
    - 子任务
    - 难点
    - 识别
- 5.关系抽取

# 1.信息提取

## （1）信息提取定义
先将自然语言处理句子这样的非结构化数据转化为结构化数据，然后利用强大的查询工具
比如SQL。这种从文本获取意义的方法被称为信息提取。

## （2）信息提取流程

![](http://i.imgur.com/NzOMEFX.png)

前三步分别是：分句、分词、词性标注。

*代码可参考*:
```python
import nltk
def ie_preprocess(document):
    sentences = nltk.sent_tokenize(document)
    sentences =[nltk.word_tokenize(sent) for sent in sentences]
    sentences = [nltk.pos_tag(sent) for sent in sentences]
```
# 2.分块（chunking）
分块是进行实体识别前的基本操作，因为对于一个实体，它很有可能是由多个单词组成。

## （1）动名词分块（牛刀小试）

我们先用正则实现一个最简单的分块器。

我们考虑名词短语分块（NP-chunking）。NP-chunking最有用的来源是词性标记。我们
定义一个规则：一个NP块由一个可选的限定词（DT）后面跟着任意数目的形容词（JJ）然后是一个
名词（NN）组成。
```python
# -*-coding: utf-8 -*-
import nltk
sentence = [("the", "DT"), ("little", "JJ"), ("yellow", "JJ"), ("dog", "NN"), ("barked", "VBD"), ("at", "IN"),
            ("the", "DT"), ("cat", "NN")]
grammar = 'NP: {<DT>?<JJ>*<NN>}'  # 标记模式，类似于正则但不是正则！
cp = nltk.RegexpParser(grammar)
result = cp.parse(sentence)
print result
```
*output*

    (S
        (NP the/DT little/JJ yellow/JJ dog/NN)
        barked/VBD
        at/IN
        (NP the/DT cat/NN))

## （2）用正则表达式chunk

对于（1）我们再增加两个规则
```python
# -*-coding: utf-8 -*-
import nltk

# rule1:一个可选的限定词或所有格代名词，零个或多个形容词，然后跟一个名词
# rule2:一个或多个专有名词
grammar = r"""
    NP: {<DT|PP\$>?<JJ>*<NN>}
        {<NNP>+}
"""
cp = nltk.RegexpParser(grammar)
sentence = [("Rapunzel", "NNP"), ("let", "VBD"), ("down", "RP"), ("her", "PP$"), ("long", "JJ"), ("golden", "JJ"),
            ("hair", "NN")]
print cp.parse(sentence)  # 结果为树状结果，你可以掉用draw()方法plot出来；还可以迭代它
```
*output*

    (S
        (NP Rapunzel/NNP)
        let/VBD
        down/RP
        (NP her/PP$ long/JJ golden/JJ hair/NN))

当出现标记模式匹配位置重叠，最左边的匹配优先。比如：`金融` `市场` `投资`，一种模式既可以识别
`金融市场`，也可以识别`市场投资`,则优先匹配`金融市场`。

## （3）chink（加缝隙）
chink指从一个chunk中去除一个标识符序列。

*chink的三种情况*

![](http://i.imgur.com/nWhu5nd.png)

*实现chink器*
```python
grammar = r"""
    NP: {<.*>+}  # chunk everything
        }<VBD|IN>+{  # chink sequence of VBD and IN
"""
cp = nltk.RegexpParser(grammar)
```
## （4）chunk的表示：tag表示和tree表示

chunk既可以用标记来表示，也可以用树来表示。

**IOB标记表示**

在此标记中，每个标识符被用三个特殊的chunk tag标记，I（inside），O（outside），
B（begin）。下面是一个例子：
![](http://i.imgur.com/tpMShC3.png)

We是一个chunk，the yellow dog也是一个chunk。我们看到每个标识符被标记上IOB（可以加上
chunk类型后缀，比如NP）

在文件中存储chunk时，标准格式可以这么存：

    We PRP B-NP
    saw VBD O
    the DT B-NP
    ...

**tree表示**

*如下图*

![](http://i.imgur.com/dMmYyud.png)

树作为nltk中chunk的内部表示。

# 3.开发和评估分块器

## （1）读取IOB格式
一般来说，chunk类型包括NP，VP，PP。我们使用chunk.conllstr2tree()方法将IOB格式的
字符串/文件转换为树状结构。

*code*
```python
# -*-coding: utf-8 -*-
import nltk

text = '''
he PRP B-NP
accepted VBD B-VP
the DT B-NP
position NN I-NP
of IN B-PP
vice NN B-NP
chairman NN I-NP
of IN B-PP
Carlyle NNP B-NP
Group NNP I-NP
a DT B-NP
merchant NN I-NP
banking NN I-NP
concern NN I-NP
'''
tree = nltk.chunk.conllstr2tree(text, chunk_types=('NP',))
```

## （2）分块语料库
nltk提供了分块语料库，你可以使用`nltk.corpus.conll2000`访问

*code example*
```python
# -*-coding: utf-8 -*-
import nltk
from nltk.corpus import conll2000

print conll2000.chunked_sents('train.txt', chunk_types=['NP'])[0]
```

## （3）简单评估分块器
我们使用正则表达式分块器。

*我们的分块器不起作用时*
```python
# -*-coding: utf-8 -*-
import nltk
from nltk.corpus import conll2000
cp = nltk.RegexpParser('')  # 这个正则不产生任何chunk效果
test_sents = conll2000.chunked_sents('train.txt', chunk_types=['NP'])
print cp.evaluate(test_sents)
```
*output*

    ChunkParse score:
        IOB Accuracy:  44.1%
        Precision:      0.0%
        Recall:         0.0%
        F-Measure:      0.0%

IOB标记准确率超过四成表明那么多的词被标注为O，即在NP块之外。

*很简单的正则分块器*
```python
# -*-coding: utf-8 -*-
import nltk
from nltk.corpus import conll2000
grammar = r'NP: {<[CDJNP].*>+}'  # 以名词短语的特征字母（如CD，DT，JJ）开头的标记（？？？）
cp = nltk.RegexpParser(grammar)
test_sents = conll2000.chunked_sents('train.txt', chunk_types=['NP'])
print cp.evaluate(test_sents)
```
*output*

    ChunkParse score:
        IOB Accuracy:  87.4%
        Precision:     69.7%
        Recall:        67.5%
        F-Measure:     68.6%

## （4）使用unigram标注器建立分块器
使用unigram标注器可以标注词性，同样的道理，我们可以设想是否可以用unigram标注器
建立分块器。（注意：unigram只是从训练集记住那些已经有tag的词，它不具备泛化能力）
```python
# -*-coding: utf-8 -*-
import nltk
from nltk.corpus import conll2000

# 建立unigram分块器
class UnigramChunker(nltk.ChunkParserI):
    def __init__(self, train_sents):
        train_data = [
            [(tag, chunk_tag) for word, tag, chunk_tag in nltk.chunk.tree2conlltags(sent)] for sent in train_sents
        ]
        self.tagger = nltk.UnigramTagger(train_data)

    def parse(self, sentence):
        pos_tags = [pos for (word, pos) in sentence]  # 词性tag
        tagged_pos_tags = self.tagger.tag(pos_tags)
        chunktags = [chunktag for (pos, chunktag) in tagged_pos_tags]
        conlltags = [(word, pos, chunktag) for ((word, pos), chunktag) in zip(sentence, chunktags)]
        return nltk.chunk.conlltags2tree(conlltags)
# 测试
test_sents = conll2000.chunked_sents('test.txt', chunk_types=['NP'])
train_sents = conll2000.chunked_sents('train.txt', chunk_types=['NP'])
unigram_chunker = UnigramChunker(train_sents)
print unigram_chunker.evaluate(test_sents)
```
*output*

    ChunkParse score:
        IOB Accuracy:  92.9%
        Precision:     79.9%
        Recall:        86.8%
        F-Measure:     83.2%

注意：

（1）建立bigram chunker和unigram chunker几乎一样，只不过把UnigramTagger换成BigramTagger。理论上讲性能稍微优于unigram chunker

（2）n-gram chunker的chunk完全基于词性。

## （5）训练基于分类器的分块器
分类器分配IOB标记给句子中的每个词，然后我们可以利用这些IOB标记，得出分块。

很类似我们在词性标注任务中的方法。

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
from nltk.corpus import conll2000


class ConsecutiveNPChunkTagger(nltk.TaggerI):
    def __init__(self, train_sents):
        train_set = []
        for tagged_sent in train_sents:
            untagged_sent = nltk.tag.untag(tagged_sent)
            history = []
            for i, (word, tag) in enumerate(tagged_sent):
                featureset = npchunk_features(untagged_sent, i, history)
                train_set.append((featureset, tag))
                history.append(tag)
        self.classifier = nltk.NaiveBayesClassifier.train(train_set)

    def tag(self, sentence):
        history = []
        for i, word in enumerate(sentence):
            featureset = npchunk_features(sentence, i, history)
            tag = self.classifier.classify(featureset)
            history.append(tag)
        return zip(sentence, history)


class ConsecutiveNPChunker(nltk.ChunkParserI):
    def __init__(self, train_sents):
        tagged_sents = [
            [((w, t), c) for (w, t, c) in nltk.chunk.tree2conlltags(sent)] for sent in train_sents
        ]
        self.tagger = ConsecutiveNPChunkTagger(tagged_sents)

    def parse(self, sentence):
        tagged_sents = self.tagger.tag(sentence)
        conlltags = [(w, t, c) for ((w, t), c) in tagged_sents]
        return nltk.chunk.conlltags2tree(conlltags)


def npchunk_features(sentence, i, history):
    word, pos = sentence[i]
    if i == 0:
        prev_word, prevpos = '<START>', '<START>'
    else:
        prev_word, prevpos = sentence[i-1]
    return {'pos': pos, 'prev_pos': prevpos, 'prev_word': prev_word}  # 将本单词词性、前一个单词、前一个单词词性作为特征


test_sents = conll2000.chunked_sents('test.txt', chunk_types=['NP'])
train_sents = conll2000.chunked_sents('train.txt', chunk_types=['NP'])
chunker = ConsecutiveNPChunker(train_sents)
print chunker.evaluate(test_sents)
```
*output*

    ChunkParse score:
        IOB Accuracy:  93.5%
        Precision:     80.9%
        Recall:        87.9%
        F-Measure:     84.2%

接下来，如果想提高分类器的能力，那就试着选取好一点的特征。

# 4.命名实体识别（NER）
NER有两个**子任务**：确定NE的边界及其类型。它是信息提取中关系识别的前提。实体类型
一般包括：组织、人员、地点、日期、时间、货币、GPE。

NER识别的**难点**在于：

（1）使用名称词典查找来识别命名实体是不可能的，因为数据量实在太大，而且更新太快

（2）许多NE措辞都有歧义

（3）会有多词NE和包含其它NE的NE

NER非常适合用分类来处理，我们可以自己训练一个分类器来做，NLTK也已经训练好了一个
。下面介绍NLTK中NER的实现。

*文件中数据是这么存储*

    Eddy N B-PER
    Bonte N I-PER
    is V O
    woordvoerder N O
    van Prep O
    diezelfde Pron O
    Hogeschool N B-ORG
    . Punc O

我们可以用`nltk.chunk.conlltags2tree()`函数将标记序列转化为一个chunk tree。
我们可以通过`nltk.ne_chunk()`使用训练好的NER器.如果参数`binary`设置为`True`,
命名实体只被标注为`NE`,否则分类器会添加类型标签，如`PERSON`，`ORGANIZATION`，`GPE`

*code examples*
```python
# -*-coding: utf-8 -*-
import nltk

sent = nltk.corpus.treebank.tagged_sents()[22]
print nltk.ne_chunk(sent, binary=True)
print u'---------------------------------------------------------------------------'
print nltk.ne_chunk(sent)

```
*output*

    (S
     The/DT
     (NE U.S./NNP)
     is/VBZ
     one/CD
     of/IN
     the/DT
     few/JJ
     industrialized/VBN
     nations/NNS
     that/WDT
     *T*-7/-NONE-
     does/VBZ
     n't/RB
     have/VB
     a/DT
     higher/JJR
     standard/NN
     of/IN
     regulation/NN
     for/IN
     the/DT
     smooth/JJ
     ,/,
     needle-like/JJ
     fibers/NNS
     such/JJ
     as/IN
     crocidolite/NN
     that/WDT
     *T*-1/-NONE-
     are/VBP
     classified/VBN
     *-5/-NONE-
     as/IN
     amphobiles/NNS
     ,/,
     according/VBG
     to/TO
     (NE Brooke/NNP)
     T./NNP
     Mossman/NNP
     ,/,
     a/DT
     professor/NN
     of/IN
     pathlogy/NN
     at/IN
     the/DT
     (NE University/NNP)
     of/IN
     (NE Vermont/NNP College/NNP)
     of/IN
     (NE Medicine/NNP)
     ./.)
    ---------------------------------------------------------------------------
    (S
     The/DT
     (GPE U.S./NNP)
     is/VBZ
     one/CD
     of/IN
     the/DT
     few/JJ
     industrialized/VBN
     nations/NNS
     that/WDT
     *T*-7/-NONE-
     does/VBZ
     n't/RB
     have/VB
     a/DT
     higher/JJR
     standard/NN
     of/IN
     regulation/NN
     for/IN
     the/DT
     smooth/JJ
     ,/,
     needle-like/JJ
     fibers/NNS
     such/JJ
     as/IN
     crocidolite/NN
     that/WDT
     *T*-1/-NONE-
     are/VBP
     classified/VBN
     *-5/-NONE-
     as/IN
     amphobiles/NNS
     ,/,
     according/VBG
     to/TO
     (PERSON Brooke/NNP T./NNP Mossman/NNP)
     ,/,
     a/DT
     professor/NN
     of/IN
     pathlogy/NN
     at/IN
     the/DT
     (ORGANIZATION University/NNP)
     of/IN
     (PERSON Vermont/NNP College/NNP)
     of/IN
     (GPE Medicine/NNP)
     ./.)

# 5.关系抽取

一旦文本中的命名实体被识别，我们就可以提取它们之间存在的关系。我们通常会找
指定类型的命名实体之间的关系。其中一种常用的方法是首先寻找所有的`（X， a，Y）`
形式的三元组，其中`X`和`Y`是指定类型的命名实体,`a`表示`X`和`Y`之间关系的字符串.
然后我们可以使用正则表达式从`a`的中抽出我们正在查找的关系。

*code example*

```pyhton
# -*-coding: utf-8 -*-
import nltk
import re

IN = re.compile(r'.*\bin\b(?!\b.+ing)')  # 抽取包含in的关系
for doc in nltk.corpus.ieer.parsed_docs('NYT_19980315'):
    for rel in nltk.sem.extract_rels('ORG', 'LOC', doc, corpus='ieer', pattern=IN):
        print nltk.sem.rtuple(rel)
```
*output*

    [ORG: u'WHYY'] u'in' [LOC: u'Philadelphia']
    [ORG: u'McGlashan &AMP; Sarrail'] u'firm in' [LOC: u'San Mateo']
    [ORG: u'Freedom Forum'] u'in' [LOC: u'Arlington']
    [ORG: u'Brookings Institution'] u', the research group in' [LOC: u'Washington']
    [ORG: u'Idealab'] u', a self-described business incubator based in' [LOC: u'Los Angeles']
    [ORG: u'Open Text'] u', based in' [LOC: u'Waterloo']
    [ORG: u'WGBH'] u'in' [LOC: u'Boston']
    [ORG: u'Bastille Opera'] u'in' [LOC: u'Paris']
    [ORG: u'Omnicom'] u'in' [LOC: u'New York']
    [ORG: u'DDB Needham'] u'in' [LOC: u'New York']
    [ORG: u'Kaplan Thaler Group'] u'in' [LOC: u'New York']
    [ORG: u'BBDO South'] u'in' [LOC: u'Atlanta']
    [ORG: u'Georgia-Pacific'] u'in' [LOC: u'Atlanta']

