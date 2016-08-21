### 目录

- 1.信息提取


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
> (S
>
>  (NP the/DT little/JJ yellow/JJ dog/NN)
>
>  barked/VBD
>
>  at/IN
>
>  (NP the/DT cat/NN))

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
> (S
>
>  (NP Rapunzel/NNP)
>
>  let/VBD
>
>  down/RP
>
>  (NP her/PP$ long/JJ golden/JJ hair/NN))

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
