目录
1. 自然语言理解
    - 将语言翻译为SQL
    - 语义的逻辑表示
2. 命题逻辑
3. 一阶逻辑




# 1.自然语言理解

## （1）将语言翻译为SQL
基于特征的文法形式可以很容易将英语翻译为SQL（具体的例子这里省略）。主要有两点
弊端：
1. 整个句子的SQL翻译是由句子成分的翻译建立起来的。然而这些成分的意思并没有很多的合理性。
2. 我们“硬生生”地把一些数据库细节加入文法。

这些因素表明：*我们需要将语言翻译成比SQL更加抽象和通用的东西*。接下去的几节将会探讨
将语言翻译成*逻辑*，大多数通过自然语言查询数据库的重要尝试都是使用这种方法。

## （2）语义的逻辑表示
自然语言语义表示的*基于逻辑的方法*关注*那些指导我们判断自然语言的一致性和不一致性方面*。

# 2.命题逻辑
设计一种逻辑语言的目的是使推理形式更加明确。因此，它可以在自然语言中确定一组句子是否一致
。命题逻辑使我们能只表示语言结构的对应与句子的特定连接词的那部分。比如`[Klaus chased Evi] and [Evi ran away].`
我们可以用`Φ`和`Ψ`替代两个子句，用`&`替代`and`,则句子可以表示为：`Φ & Ψ`。
其它的连接词还有：`not`、`or`、`if … then …`，这些连词称为布尔运算符。

*查看nltk提供的操作符*
```python
# -*- coding: utf-8 -*-
import nltk
print nltk.boolean_ops()

```
*output*
```
negation       	-
conjunction    	&
disjunction    	|
implication    	->
equivalence    	<->
```
用命题符号和布尔运算符，我们可以建立命题逻辑的规范公式的无限集合。
`iff`即`if and only if`

![真值条件](http://i.imgur.com/l3RoGSp.png)

我们来完成一个推理：
```
Sylvania is to the north of Freedonia.
Therefore, Freedonia is not to the north of Sylvania.
```
我们使用`SnF`表示命题`Sylvania is to the north of Freedonia`，用`FnS`表示命题
`Freedonia is to the north of Sylvania`，用`-FnS`表示命题`Freedonia is not to the north of Sylvania`。

`to the north of`是一个非对称关系,所以我们要证明的推理的前提`Sylvania is to the north of Freedonia`
包含了两个命题，`SnF, SnF -> -FnS`。我们的证明用推理式写出:
```
[SnF, SnF -> -FnS] / -FnS
```
*用nltk推理*
```python
# -*- coding: utf-8 -*-
import nltk
read_expr = nltk.sem.Expression.fromstring
SnF = read_expr('SnF')
NotFnS = read_expr('-FnS')
R = read_expr('SnF -> -FnS')
prover = nltk.Prover9()
prover._binary_location = r'D:\LADR1007B-win\Prover9\bin'  # Prover9可执行程序路径
print prover.prove(NotFnS, [SnF, R])
```
*output*
```
True
```

<br />

**一个命题逻辑模型**

目标：为每个公式分配布尔值

过程：首先，为每个命题符号分配布尔值，然后确定布尔运算符的含义和运用它们到这些
公式的组件的值，来计算复杂公式的值。

*code example*
```python
# -*- coding: utf-8 -*-
import nltk
val = nltk.Valuation([('P', True), ('Q', True), ('R', False)])  # 结果为dict形式
dom = set([])
g = nltk.Assignment(dom)
model = nltk.Model(dom, val)
print model.evaluate('(P & Q)', g)
print model.evaluate('-(P & Q)', g)
print model.evaluate('(P & R)', g)
print model.evaluate('(P | Q)', g)
```
*output*
```
True
False
False
True
```

# 3.一阶逻辑
上节我们我们使用命题逻辑建模，它不能深入句子内部，而本节所讲的一阶逻辑可以。