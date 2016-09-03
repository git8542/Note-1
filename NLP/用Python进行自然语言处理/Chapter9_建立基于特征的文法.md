### 目录
- 1.文法特征
    - 句法协议
    - 使用特征约束
    - 特征值的类型
- 2.处理特征结构
    - 特征结构基本操作
    - 特征结构的图形化表示
    - 包含和统一


### Overview
为了获得更大的灵活性，我们改变对待文法类别（`S`、`NP`、`V`等）的方式，将它们分解为
类似字典的结构，以便于可以提取一系列的值作为特征。

本章要解决以下三个问题：
- 我们如何使用特征扩展上下文无关文法框架，以便获得更细粒度的对文法类别和产生式的
控制？
- 特征结构的主要形式化属性是什么，我们如何使用它们来计算？
- 我们使用基于特征的文法能捕捉到什么语言模式和文法结构？

# 1.文法特征

## （1）句法协议（agreement）
在英语中，动词的形态属性与主语名词短语的属性一起变化，这种一起变化被称为**协议**。
比如：动词与它的主语在人称和数量上保持一致。（动词的第三人称单数时态对应着主语也是
单三）

我们依然使用之前介绍的CFG编码：
- 假设我们忽略*句法协议规则*，则我们可能会得到`The dog run`(缺s)这种错误的句子。（粒度不够）
- 假设我们为了使用这些*句法协议规则*，扩展了文法类别：比如我们将类别`V`扩展为`V_single`
和`V_plural`,将`N`扩展为`N_single`和`N_plural`。这种类别扩展会导致其左端类别跟着扩展
，比如对于之前的产生式`NP -> Det N`，随着`N`的扩展,`NP`也要跟着扩展，使最后的产生式
数量增加了数倍。这使得这种方法并不被看好。

## （2）使用特征约束

语言类别具有属性，比如名词具有复数的属性。我们可以这么表述一个复数名词`N[NUM=pl]`
,这里`NUM`是类别`N`的一个文法特征，含义是`单复数`;这里`pl`是特征值，表示`复数`。

<br />

我们可以使用特征值变量，并利用这些表示限制。
- 对于产生式： `S -> NP[NUM=?n] VP[NUM=?n]`，
我们使用`?n`作为`NUM`值上的变量，它可以在给定的产生式中被实例化为`sg`或`pl`。这
条产生式是说：不管`NP`的`NUM`特征取何值，`VP`的`NUM`也必须取相应的值。
- 对于产生式：`VP[NUM=?n] -> V[NUM=>n]`，核心动词的`NUM`必须与它的`VP`父亲的
`NUM`值相同

<br />

对于某些限定词,比如`the`，它不像`this、these`,它对与它结合的名词文法上的数量并不
挑剔。对于以它为右侧的产生式，我们怎么写才最简便呢？
- 最复杂的方式:`Det[NUM=sg] -> 'the'` `Det[NUM=pl] -> 'the'`
- 简化一点:`Det[NUM=?n] -> 'the'`
- 最简:`Det -> 'the'`,`NUM`特征就不要写了。

<br />

**几点注意**：

1. 一个句法类型可以有多个特征，例如`V[TENSE=pres, NUM=pl]`
2. 把产生式放在一个文件中我们编辑、测试、修改都是很方便的
3. 若产生式出现`%start S`时，表明分析器应该以`S`作为文法开始符号

<br />

*使用基于特征的图表分析器*
```python
# -*- coding: utf-8 -*-
import nltk
from nltk import load_parser
tokens = 'Kim likes children'.split()
feature_chart_parser = load_parser('grammars/book_grammars/feat0.fcfg', trace=0)
trees = feature_chart_parser.parse(tokens)
for tree in trees:
    print tree
```
*output*

    (S[]
      (NP[NUM='sg'] (PropN[NUM='sg'] Kim))
      (VP[NUM='sg', TENSE='pres']
        (TV[NUM='sg', TENSE='pres'] likes)
        (NP[NUM='pl'] (N[NUM='pl'] children))))

## （3）特征值的类型
特征值的类型可以分为**原子类型**和**子特征结构类型**。

<br />

**原子类型**

像`sg` `pl`这种类型的特征值，不可分为更小的部分，被称为原子类型。特殊的原子类型
是*布尔类型*。 `V[aux=+] -> 'can'`表示`can`是助动词（`aux`特征表示`是否助动词`）
,更为提倡的写法`V[+aux] -> 'can'`

<br />

**子特征结构类型**

特征值本身就是特征结构。e.g. 我们可以将`人称`、`数量`、`性别`组合在一起，表示为
`AGR`的值。

*表示为特征值矩阵（AVM）形式*

![AVM](http://i.imgur.com/AVobQEo.png)

*在NLTK中的表示*

    [POS = N         ]
    [                ]
    [AGR = [PER = 3 ]]
    [      [NUM = pl ]]
    [      [GND = fem ]]

*使用范例*
```
S -> NP[AGR=?n] VP[AGR=?n]
NP[AGR=?n] -> PropN[AGR=?n]
VP[TENSE=?t, AGR=?n] -> Cop[TENSE=?t, AGR=?n] Adj
Cop[TENSE=pres, AGR=[NUM=sg, PER=3]] -> 'is'
PropN[AGR=[NUM=sg, PER=3]] -> 'Kim'
Adj -> 'happy'
```
<br />

**AVM**

    Attribute-value matrices are a practical way to present feature
    structures as a group of attributes and their corresponding values.

# 2.处理特征结构
本节主要关注上节所讲的特征结构在nltk中的操作。

## (1)特征结构基本操作

**特征结构的创建**

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
# 方法1：直接输入kv对创建
print 'Method 1'
print nltk.FeatStruct(TENSE='past', NUM='sg')
# 方法2：字符串创建
print 'Method 2'
print nltk.FeatStruct("[TENSE='past', NUM='sg']")

```
*output*
```
Method 1
[ NUM   = 'sg'   ]
[ TENSE = 'past' ]
Method 2
[ NUM   = 'sg'   ]
[ TENSE = 'past' ]
```

<br />

**索引特征值**

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
fs = nltk.FeatStruct(TENSE='past', NUM='sg')
print fs['TENSE']

```
*output*
```
past
```

<br />

**创建特征值为特征结构的特征结构**

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
fs1 = nltk.FeatStruct(PER=3, NUM='pl', GND='fem')
fs2 = nltk.FeatStruct(POS='N', AGR=fs1)
print fs2

```
*output*
```
[       [ GND = 'fem' ] ]
[ AGR = [ NUM = 'pl'  ] ]
[       [ PER = 3     ] ]
[                       ]
[ POS = 'N'             ]
```

## (2)特征结构的图形化表示
特征结构可以作为*有向无环图（DAGs）*来表示：特征名称作为弧上的标签，特征值作为弧指向节点的
标签。

*一个简单的图*

![DAGS1](http://i.imgur.com/ceI3Cns.png)

<br />

*带有特征值为特征结构的图*

![DAGS2](http://i.imgur.com/0TFT6lP.png)

标签的元组可以表示路径：`(address, street)`就是一个*特征路径*。

<br />

**共享**

*原始图*

![Ori](http://i.imgur.com/Qx36Crl.png)

我们发现最下面两棵子树其实是完全一样的，所以我们只保留一份。

*共享后的图*

![Share](http://i.imgur.com/09Iau1y.png)

这种操作被称为*结构共享*或*重入*

<br />

**共享在nltk中的表示**

为了表示*重入*，我们将会在共享特征结构第一次出现的地方加一个括号括起来的
数字前缀，例如`(1)`。以后任何对这个结构的引用将使用符号`->(1)`。

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
print nltk.FeatStruct("""[NAME='Lee', ADDRESS=(1)[NUMBER=74, STREET='rue Pascal'],
    SPOUSE=[NAME='Kim', ADDRESS->(1)]]""")

```
*output*
```
[ ADDRESS = (1) [ NUMBER = 74           ] ]
[               [ STREET = 'rue Pascal' ] ]
[                                         ]
[ NAME    = 'Lee'                         ]
[                                         ]
[ SPOUSE  = [ ADDRESS -> (1)  ]           ]
[           [ NAME    = 'Kim' ]           ]
```

*注意*

括号内的整数被称为*标记*或*同指标志*（coindex）。整数的选择并不重要，一个
特征结构中可以出现任意个标记。

## (3)包含和统一

**包含**

下面的 a 比 b 更一般， b 比 c 更一般
```
a. [NUMBER = 74]

b. [NUMBER = 74          ]
   [STREET = 'rue Pascal']

c. [NUMBER = 74          ]
   [STREET = 'rue Pascal']
   [CITY = 'Paris'       ]
```
这个顺序被称为*包含（subsumption）*:一个更一般的特征结构包含一个较少一般的。
如果`FS0`包含`FS1`，那么`FS1`必须具备`FS0`的所有路径和等价路径，也可能有额外的路径
和等价路径。

<br />

**统一**

合并两个特征结构的信息被称为*统一*，由方法`unify()`支持。

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
fs1 = nltk.FeatStruct(NUMBER=74, STREET='rue Pascal')
fs2 = nltk.FeatStruct(CITY='Paris')
print fs1.unify(fs2)

```
*output*
```
[ CITY   = 'Paris'      ]
[ NUMBER = 74           ]
[ STREET = 'rue Pascal' ]
```
*注意*
1. 统一是对称的,`fs1.unify(fs2)`和`fs2.unify(fs1)`等价。
2. 如果我们统一两个具有包含关系的特征结构，那么结果将是两个中更具体的那个。
3. 如果两个特征结构具有相同的路径`PI`，但路径所指向的值却不相同，那么统一
之后的结果将是`None`

可以将*统一*和*结构共享*结合起来，用来给特征结构的较深层添加特征。

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
fs1 = nltk.FeatStruct("""[NAME=Lee,
                          ADDRESS=[NUMBER=74,STREET='rue Pascal'],
                          SPOUSE= [NAME=Kim,ADDRESS=[NUMBER=74,STREET='rue Pascal']]]""")
fs2 = nltk.FeatStruct("[SPOUSE=[ADDRESS=[CITY='Paris']]]")
fs = fs1.unify(fs2)
print 'Before unify:'
print fs1
print '\nLater unify:'
print fs

```
*output*
```
Before unify:
[ ADDRESS = [ NUMBER = 74           ]               ]
[           [ STREET = 'rue Pascal' ]               ]
[                                                   ]
[ NAME    = 'Lee'                                   ]
[                                                   ]
[           [ ADDRESS = [ NUMBER = 74           ] ] ]
[ SPOUSE  = [           [ STREET = 'rue Pascal' ] ] ]
[           [                                     ] ]
[           [ NAME    = 'Kim'                     ] ]

Later unify:
[ ADDRESS = [ NUMBER = 74           ]               ]
[           [ STREET = 'rue Pascal' ]               ]
[                                                   ]
[ NAME    = 'Lee'                                   ]
[                                                   ]
[           [           [ CITY   = 'Paris'      ] ] ]
[           [ ADDRESS = [ NUMBER = 74           ] ] ]
[ SPOUSE  = [           [ STREET = 'rue Pascal' ] ] ]
[           [                                     ] ]
[           [ NAME    = 'Kim'                     ] ]
```
实际上我们可以直接向在深层嵌套的字典添加key-value那样操作。
(尽管书上没有介绍，我倒觉得这样更方便)

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
fs1 = nltk.FeatStruct("""[NAME=Lee,
                          ADDRESS=[NUMBER=74,STREET='rue Pascal'],
                          SPOUSE= [NAME=Kim,ADDRESS=[NUMBER=74,STREET='rue Pascal']]]""")
fs1['SPOUSE']['ADDRESS']['CITY'] = 'Paris'
```

*结构共享也可以使用变量`?x`来表示*

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
fs1 = nltk.FeatStruct("[ADDRESS1=[NUMBER=74, STREET='rue pascal']]")
fs2 = nltk.FeatStruct("[ADDRESS1=?x, ADDRESS2=?x]")
print fs1.unify(fs2)

```
*output*
```
[ ADDRESS1 = (1) [ NUMBER = 74           ] ]
[                [ STREET = 'rue pascal' ] ]
[                                          ]
[ ADDRESS2 -> (1)                          ]
```
