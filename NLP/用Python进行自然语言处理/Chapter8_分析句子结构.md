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
    - 递归下降分析
    - 移进规约分析
    - 左角落分析

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

## （2）移进规约分析

## （3）左角落分析


