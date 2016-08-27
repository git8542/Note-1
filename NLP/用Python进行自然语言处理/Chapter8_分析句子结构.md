### 目录
- 1.一些语法困境
- 2.成分结构
- 3.上下文无关文法
    - 什么是CFG
    - 使用nltk定义CFG
    - 从文件加载文法
    - 句法结构中的递归
    - GUI查看递归下降过程
- 4.上下文无关文法分析
    - 递归下降分析（RDP）
    - 移进归约分析（SRP）
    - 左角落分析（LCP）
    - 符合语句规则的子串表
- 5.依存文法

### Overview
前面我们一直关注词，现在我们来看看句子。`语言`被认为是所有合乎文法的句子的巨大集合
，一个`文法`是一个形式化符号，可用于产生这个集合的成员。
# 1.一些语法困境
本书提到了三种语法困境：

    （1）语言数据并不是越多越好，因为其中可能包含一些错误语法
    （2）句子可能会存在很多层嵌套
    （3）句子可能会存在歧义

# 2.成分结构
成分结构基于对词和其他词结合在一起形成单元的观察。一个词序列形成这样一个单元的
证据是它是可替代的——也就是说，在一个符合语法规则的句子中的词序列可以被一个更
小的序列替代而不会导致句子不符合语法规则。思考以下句子：
> The little bear saw the fine fat trout in the brook.

我们可以替换`The little bear`为`He`,使句子仍符合语法，这表明`The little bear`是一个
单元。

*实际上，形成一个单元的每个词序列可以被一个单独的词替换，使得最终只有两个元素*
![](http://i.imgur.com/x5owK48.png)

*我们可以画出句子的语法树*
![](http://i.imgur.com/v7eOHVx.png)

# 3.上下文无关文法（context-free grammar，CFG）

## （1）什么是CFG
A context-free grammar (CFG) is a term used in formal language theory to
describe a certain type of formal grammar. A context-free grammar is a set
 of production rules that describe all possible strings in a given formal
 language. Production rules are simple replacements.For example, the rule
`A-->a` Replaces `A` with `a` . There can be multiple replacement rules
for any given value. For example,`A-->a` `B-->b` means that
`A`can be replaced with either `a` or `b`.

In context-free grammars, all rules are one to one, one to many, or one
to none. These rules can be applied regardless of context. The left-hand
side of the production rule is also always a nonterminal symbol. This
means that the symbol does not appear in the resulting formal language.
Rules can also be applied in reverse to check if a string is grammatically
 correct according to the grammar.

(From wikipedia)
## （2）使用nltk定义CFG
第一条产生式左端是文法的开始符号，通常是`S`.我们来看一个例子：

*example*
```python
# -*- coding: utf-8 -*-
import nltk
__author__ = 'BrownWong'
grammar = nltk.CFG.fromstring("""
    S -> NP VP
    VP -> V NP | V NP PP
    PP -> P NP
    V -> "saw" | "ate" | "walked"
    NP -> "John" | "Mary" | "Bob" | Det N | Det N PP
    Det -> "a" | "an" | "the" | "my"
    N -> "man" | "dog" | "cat" | "telescope" | "park"
    P -> "in" | "on" | "by" | "with"
    """)
sent = "Mary saw Bob".split()
rd_parser = nltk.RecursiveDescentParser(grammar)
for tree in rd_parser.parse(sent):
    print tree

```
*output*

    (S (NP Mary) (VP (V saw) (NP Bob)))

*附句法类型表*

符号|意思
----|----
S   |句子
NP  |名词短语
VP  |动词短语
PP  |介词短语
Det |限定词
N   |名词
V   |动词
P   |介词

注意：

（1）`|`表示缩写，如：`VP -> V NP | V NP PP`是`VP -> V NP`和`VP -> V NP PP`
的缩写。

（2）当句子存在歧义时，一个句子可能产生多棵树。

## （3）从文件加载文法
使用以下语句:

    grammer = nltk.data.load('file:mygrammer.cfg')

注意：

（1）请文件格式（文件名后缀）正确。

（2）不能将一个文法类型和一个词汇写在同一个产生式的右侧。如：`PP -> 'of' NP`
不被允许。

（3）不能在产生式右侧出现多个词的词汇项。不能写成`NP -> 'New York'`这种形式
，而要写成`NP -> 'New_York'`这种形式。

## （4）句法结构中的递归

**直接递归**：形如`Nom -> Adj Nom`这种形式的产生式称为直接递归。

**间接递归**：形如`S -> NP VP`,`VP -> V S`这种形式的产生式称为间接递归。

## （5）GUI查看递归下降过程
使用`nltk.app.rdparser()`打开GUI工具。
# 4.上下文无关文法分析
## （1）递归下降分析
一种自上而下的分析

*递归下降分析的过程如下图*：

![](http://i.imgur.com/igMfs1T.png)

*用nltk实现*
```python
rd_parser = nltk.RecursiveDescentParser(grammar)
for tree in rd_parser.parse(sent):
    print tree
```
*递归下降分析的缺点*

    （1）左递归产生式，如`NP -> NP PP`，会进入死循环。
    （2）分析器浪费了很多时间处理不符合输入句子的词和结构。
    （3）回溯过程会丢弃分析过的成分，它们可能会在之后重建。比如：从`VP -> V NP`
    上回溯将放弃为NP创建的子树。如果分析器之后将处理`VP -> V NP PP`，那NP子树必须重建

## （2）移进归约分析（shift reduce）
**基本思想**

移进归约分析器尝试找到对应文法产生式右侧的词和短语序列，用左侧的替换它们，直到
整个句子被归约为一个`S`。

*移进归约分析的过程如下图*
![](http://i.imgur.com/SWQSnLa.png)

*GUI查看过程*

使用`nltk.app.srparse()`函数

*code example*

```python
sr_parse = nltk.ShiftReduceParser(grammar)
sent = 'Mary saw a dog'.split()
for tree in sr_parse.parse(sent):
    print tree
```

**评价**

分析器经常会面临两个选择：（a）有多种归约时，应该选择哪种归约（b）当移进和归约
都可以时选择哪个

*缺点*：可能会到达一个死胡同，而不能找到任何解析，尽管输入的句子是合法的。当这种
情况发生时，没有剩余的输入，而堆栈包含不能被归约到一个`S`的项目。原因是：较早
做出的选择不能被分析器撤销。

*优点*：对比递归下降分析器，（a）它只建立与输入中词对应的结构（b）每个结构只建立
一次
## （3）左角落分析（Left Corner Parsing）
**基本思想**：

递归下降分析器当遇到左递归产生式时，会进入无限循环。这是由于它盲目应用文法产生式
而不考虑实际输入的句子。

左角落分析器是自下而上与自上而下方法的混合体。准确来讲，是一个带自下而上过滤的
自上而下的分析器。它会维护一个左角落表。分析器每次考虑产生式时，它会检查下一个
输入词是否与左角落表格中至少一种非终结符的类别相容。

**LCP过程演示如下**：

[9.1.3 Combining Top-down and Bottom-up Information](http://cs.union.edu/~striegnk/courses/nlp-with-prolog/html/node53.html#l7.sec.leftcorner)

*code example*
```python
lr_parse = nltk.LeftCornerChartParser(grammar)
for tree in lr_parse.parse(sent):
    print tree
```
## （4）符合语法规则的子串表
这种方法利用动态规划存储中间结果，在适当的时候重用它们，能显著提高效率。动态规划
使我们只用一次建立`PP in my pajamas`之后，将其存入一个表格，然后在我们需要作为
对象`NP`或者更高的`VP`的组成成分用到它时，我们就查找表格。这个表格被称为*符合语法规则的子串表*
或者简称为WFST。

对于`I shot an elephant in my pajamas`这个句子，我们看看它的WFST

*一个WFST的例子*

![](http://i.imgur.com/7YgvKX0.png)

*它的Graph表示*

![](http://i.imgur.com/Xfwmruw.png)

在WFST中，我们通过填充三角矩阵中的单元记录词的位置：纵轴表示一个子串（词序列）的
起始位置，横轴表示一个子串的结束位置。我们在矩阵中存储词汇类型，所以`（1， 2）`
将是`shot`的词性`V`的坐标。假设对于词`an`我们有`Det`在`（2， 3）`单元
，对于词`elephant`有`N`在`（3， 4）`单元，则对于`an elephant`我们根据产生式
`NP -> Det N`可知将`NP`放入`(2, 4)`单元。

*code example*
```python
c_parse = nltk.ChartParser(grammar)  # 很多api，ChartParser只是其中之一
for tree in c_parse.parse(sent):
    print tree
```

**WFST缺点**

    （1）WFST本身不是分析树，严格地说，它是认识到一个句子被文法承认（没明白含义 Q_Q）
    （2）它要求每个非词汇文法产生式是二元的。一元会出现重叠，三元又无法表征
    （3）作为一个自下而上的方法，它潜在的存在浪费，会在不符合文法的地方提出成分。（也没明白）
    （4）它不能表示句子中的结构歧义。（？？）

# 5.依存文法

## （1）依存文法与依存关系
我们前面所讲的文法皆属于*短语结构文法*，它关注词和词序列如何结合起来形成句子成分。另一个独特的和互补的方式**依存文法**，集中
关注词和其它词之间的关系。**依存关系**是一个*中心词*和它的*依赖*之间的二元对称关系。中心词通常是动词，所有其它词要么依赖于
中心词，要么依赖路径与它联通。

依存关系可以用一个加标签的有向图表示，其中节点是词汇项，加标签的弧表示
依赖关系，方向从中心到依赖。

*NLTK表示*

NLTK为依存文法编码了一种方式，但是请注意——它只能捕捉依存关系，不能指定依存关系
类型。
```python
# -*- coding: utf-8 -*-
from nltk.grammar import DependencyGrammar
__author__ = 'BrownWong'
dep_grammar = DependencyGrammar.fromstring("""
    'shot' -> 'I' | 'elephant' | 'in'
    'elephant' -> 'an' | 'in'
    'in' -> 'pajamas'
    'pajamas' -> 'my'
    """)
print dep_grammar
```
*output*

    Dependency grammar with 7 productions
    'shot' -> 'I'
    'shot' -> 'elephant'
    'shot' -> 'in'
    'elephant' -> 'an'
    'elephant' -> 'in'
    'in' -> 'pajamas'
    'pajamas' -> 'my'
