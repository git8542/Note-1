目录

1. 自然语言理解
    - 将语言翻译为SQL
    - 语义的逻辑表示
2. 命题逻辑
3. 一阶逻辑
    - 句法
    - 一阶定理证明
    - 一阶逻辑语言总结
    - 模型的真值
    - 对独立变量赋值
    - 量化
    - 量词范围歧义
    - 建立模型



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
不是所有的自然语言语义都可以用一阶逻辑表示，但是它是计算语义的一个不错的选择，因为它具有足够的
表现力来表达语义的很多方面，而且另一方面，它有出色的现成的系统可以用来开展一阶逻辑自动推理。

## （1）句法
一阶逻辑保留所有命题逻辑的布尔运算符， 但它增加了一些重要的新机制。 首先， 命题
被分析成*谓词*和*参数*， 这将我们与自然语言的结构的距离拉近了一步。一阶逻辑的标准构造
规则承认以下术语：*独立变量*和*独立常量*、*带不同数量的参数的谓词*。

例如： `Angus walks`
可以被形式化为`walk(angus)`， `Angus sees Bertie` 可以被形式化为 `see(angus, bertie)`。 我们称
`walk`为一元谓词， `see`为二元谓词。

一阶逻辑本身没有什么实质性的关于词汇语义的表示，虽然一些词汇语义理论可以用一阶逻辑编码。
原子谓词`see(angus, bertie)`在情况`s`中是真还是假不是一个逻辑问题，而依赖于特定的估值，
即我们为常量`see`、`angus`、`bertie`选择的值。
因此这些常量被称为*逻辑常量*。相比之下，*逻辑常量*（如布尔运算符）在一阶逻辑的每个模型中的
解释总是相同的。

检查*一阶逻辑表达式的语法结构*往往是有益的，这样做的通常方式是*为表达式指定类型*。
下面介绍Montague文法规定：
```
基本类型：e是实体类型，t是公式类型（有真值的表达式的类型）
这两种类型，可以形成函数表达式的复杂类型。
例如：<e,T>是从实体到真值，即一元谓词的表达式的类型。
```
可以调用LogicParser来进行类型检查：
```python
import nltk
tlp = nltk.LogicParser(type_check=True)  # API可能已更新
parsed = tlp.parse('walk(angus)')
print parsed.argument
print parsed.argument.type
print parsed.function
print  parsed.function.type
```
*output*
```
<ConstantExpression angus>
e
<ConstantExpression walk>
<e,?>
```
注意，没有识别出`walk`的类型，因为他的类型是未知的，有可能在这个上下文中会出现别
的类型，如：`<e,e>`或`<e,<e,t>>`(二元谓词)，我们需要制定一个信号(signature)来解决这个问题：

```python
sig = {'walk': '<e, t>'}
parsed = tlp.parse('walk(angus)', sig)
parsed.function.type
```
*output*
````
<e,t>
````

<br />

*在一阶逻辑中，谓词的参数也可能是独立变量*，如`x`、`y`和`z`。e类型的变量都是小写。独立变量类似与人称代词，
如he、she和it，其中我们为了弄清楚它们的含义需要知道它们使用的上下文。

<br />

我们可以使用*存在量词*和*全称量词*绑定上面所说的变量。
```
A
a. ∃x.(dog(x) & disappear(x))  # 下述句子的公式化表述
b. At least one entity is a dog and disappeared.  # 复杂版本
c. A dog disappeared.  # 正常句子
d. exists x.(dog(x) & disappear(x))  # nltk中表述
```

```
B
a. ∀x.(dog(x) → disappear(x))  # 下述句子的公式化表述
b. Everything has the property that if it is a dog,it disappears.  # 复杂版本
c. Every dog disappeared.  # 正常句子
d. all x.(dog(x) -> disappear(x))  # nltk中表述
```
*注意*：
上面蕴含式的真值结果并不如我们所愿，当没有狗存在时，`B(a)`公式依然成立。

<br />

- 在一般情况下，变量`x`在公式`φ`中出现是自由的，如果它在`φ`中没有出现在`all x`或`exists x`范围内。
- 相反，如果x在公式φ中是受限的，那么它在`all x.φ`和`exists x.φ`限制范围内。如果公式中所
有变量都是受限的，那么我们说这个公式是封闭的。

*查看自由变量*
```
>>>lp =nltk.LogicParser()  # 旧API，新的参考上节命题逻辑使用的API
>>>lp.parse('dog(cyril)').free()
set([])
>>>lp.parse('dog(x)').free()
set([Variable('x')])
>>>lp.parse('own(angus, cyril)').free()
set([])
>>>lp.parse('exists x.dog(x)').free()
set([])
>>>lp.parse('((some x.walk(x))-> sing(x))').free()
set([Variable('x')])
>>>lp.parse('exists x.own(y,x)').free()
set([Variable('y')])
```

## （2）一阶定理证明

我们要证明下面公式:
```
all x.all y.(north_of(x, y) -> -north_of(y, x))
```
*code example*
```python
>>>NotFnS= lp.parse('-north_of(f, s)')   # 旧API
>>>SnF= lp.parse('north_of(s, f)')
>>>R= lp.parse('all x.all y.(north_of(x, y) -> -north_of(y, x))')
>>>prover= nltk.Prover9()
>>>prover.prove(NotFnS,[SnF, R])
True
>>>FnS= lp.parse('north_of(f, s)')
>>>prover.prove(FnS,[SnF, R])
False
```

## （3）一阶逻辑语言总结
我们将采取约定： `<e^n, t>`是一种由`n`个类型为`e`的参数组成产生一个类型为`t`的表达式的谓词的类型。在这
种情况下，我们说`n`是谓词的元数:
```
1. 如果P是类型<e^n, t>的谓词， α1， ... αn是e类型的术语，那么P（ α1， ...αn）的类型是t。
2. 如果α和β都是e类型，那么（ α=β）和（ α!=β）是t类型。
3. 如果φ是t类型，那么是-φ也是t类型。
4. 如果φ和ψ是t类型， 那么（ φ&ψ）、（ φ|ψ）、（ φ -> ψ） 和（ φ<->ψ） 也是t类型。
5. 如果φ是t类型， x是类型为e的变量，那么exists x.φ和all x.φ也是 t类型
```
## （4）模型的真值
我们在《句法》一小节里说过：

> 原子谓词`see(angus, bertie)`在情况`s`中是真还是假不是一个逻辑问题，而依赖于特定的估值

本节我们需要给出*一阶逻辑真值条件的语义*。

给定一阶逻辑语言`L`，`L`的模型`M`是一个`<D,Val>`对，其中`D`是一个非空集合，称为模型的*域*，`Val`是一个函数，称为*估值函数*。
估值函数按照如下的方式从`D`中分配值给`L`的表达式：

1. 对于`L`的每一个独立常量`c`，`Val(c)`是`D`中的元素
2. 对于每一个元数`n>=0`的谓词符号`P`，`Val(p)`是从`Dn`到`｛True,False｝`的函数。
（如果`P`的元数为`0`，则`Val(P)`是一个简单的真值，`P`被认为是一个命题符号;如果`P`的元
数是`2`，然后`Val(P)`将是一个从`D`的元素配对到`{True,False}`的函数）

我们将在NLTK中建立的模型中采取更方便的替代品，其中`Val(P)`是一个配对的集合`S`，定义如下：
`S = {s| f(s) = True}`这样的`f`被称为`S`的特征函数。

*NLTK的语义关系可以用标准的集合论方法表示：作为元组的集合。*

假设我们有一个域包括`Bertie`、`Olive`和`Cyril`，其中`Bertie`是男孩，`Olive`是女孩，而`Cyril`是
小狗。为了方便记述，我们用`b`、`o`和`c`作为模型中相应的标签。我们可以声明*域*如下：
```
>>>dom =set(['b', 'o', 'c'])
```
我们使用工具函数`parse_valuation()`将`“符号 => 值”`形式的字符串序列转化为一个`Valuation`
对象。
```
>>>v= """
... bertie =>b
... olive =>o
... cyril =>c
... boy=>{b}
... girl =>{o}
... dog=>{c}
... walk=>{o, c}
... see =>{(b,o),(c, b),(o, c)}
... """
>>>val = nltk.parse_valuation(v)
>>>print val
{'bertie':'b',
'boy': set([('b',)]),
'cyril': 'c',
'dog': set([('c',)]),
'girl': set([('o',)]),
'olive': 'o',
'see': set([('o', 'c'), ('c', 'b'), ('b', 'o')]),
'walk': set([('c',), ('o',)])}  # 元组形式
```

一个形式为`P(T1,... Tn)`的谓词，其中`P`是`n`元的，为真的条件是对应于`(T1, ... Tn)`的值的
元组属于`P`的值的元组的集合。
```
>>>('o', 'c') in val['see']
True
>>>('b',) in val['boy']
True
```

## （5）对独立变量赋值
在我们的模型，上下文的使用对应的是*为变量赋值*。这是一个*从独立变量到域中实体的映射*。

赋值使用构造函数`Assignment`，它也以论述的模型的*域*为参数。
```
>>>g= nltk.Assignment(dom, [('x', 'o'), ('y', 'c')])
>>>g
{'y': 'c', 'x': 'o'}
```

<br />

*为一阶逻辑的原子公式估值*
```
>>>m=nltk.Model(dom, val)
>>>m.evaluate('see(olive,y)',g)  # 使用Assignment对象作为参数
True
```

<br />

我们可以组合公式，并进行评估，来检验模型中公式的真假，这种过程称为*模型检查*。

## （6）量化
现代逻辑的关键特征就是变量满足的概念可以用来解释量化的公式。

*如果域中所有个体都满足某个公式，我们就可以使用全称量化*

使用nltk的`satisfiers()方法`，可以返回满足某个公式的所有个体集合。
```
>>>fmla2 = lp.parse('girl(x) -> walk(x)')
>>>m.satisfiers(fmla2,'x', g)  # 参数1：已分析的公式；参数2：变量；参数3：赋值
set(['c', 'b', 'o'])
```
这里，域中的所有个体都满足`girl(x) -> walk(x)`公式（注意此蕴含式为真的条件）。
我们就可以有如下*全程量化公式*
```
>>> m.evaluate('all x.(girl(x) -> walk(x))')
True
```

## （7）量词范围歧义
当我们给出一个带有两个量词的句子的形式化表述时，会发生什么？

     Everybody admires someone.
用一阶谓词至少有两种表示方法
```
a. all x.(person(x) -> exists y.(person(y) & admire(x,y)))
b. exists y.(person(y) & all x.(person(x) -> admire(x,y)))
```
这两个我们都可以使用，但是这两个的含义是不一样的：b逻辑上强于a。

## （8）建立模型

*模型的建立*

我们通过创建`Mace()`的一个实例并调用它的`build_model()`方法来调用`Mace4`模式产生器。
（与调用Prover9定理证明器类似）。一种选择是将我们的候选句子集合作为假设，保留目标为
未指定。
```
>>>a3 = lp.parse('exists x.(man(x)&walks(x))')
>>>c1 = lp.parse('mortal(socrates)')
>>>c2 = lp.parse('-mortal(socrates)')
>>>mb=nltk.Mace(5)
>>>print mb.build_model(None,[a3, c1])
True
>>>print mb.build_model(None,[a3, c2])
True
>>>print mb.build_model(None,[c1, c2])  # 创建失败（c1和c2不能同时成立）
False
```

<br />

*使用模型建立器作为定理证明器的辅助*

假设我们正试图证明·`A g`， 即`g`
是假设`A = [a1,a2, ..., an]`的逻辑派生。我们可以同样的输入提供给`Mace4`，模型建立
器将尝试找出一个反例， 就是要表明`g`不遵循从`A`。 因此， 给定此输入， `Mace4`将尝试为假
设`A`连同`g`的否定找到一个模型， 即链表`A' = [a1, a2, ..., an, -g]`。 如果`g`从`S`不能
证明出来，那么 `Mace4`会返回一个反例，比 `Prover9`更快的得出结论：*无法找到所需的证明*。
相反， 如果`g`从`S`可以证明出来， `Mace4`可能要花很长时间不能成功地找到一个反例模型，
最终会放弃。

例如这个链表：
```
[There is a woman that every man loves, Adamis a man,Eveis a woman]
```
我们的结论是：

Adam loves Eve。

Mace4能找到使假设为真而结论为假的模型吗？在下面的代码，我们使用`MaceCommand()`检查
已建立的模型。
```
>>>a4 = lp.parse('exists y. (woman(y) &all x. (man(x) -> love(x,y)))')
>>>a5 = lp.parse('man(adam)')
>>>a6 = lp.parse('woman(eve)')
>>>g= lp.parse('love(adam,eve)')
>>>mc= nltk.MaceCommand(g, assumptions=[a4, a5, a6])
>>>mc.build_model()
True
```
Mace4发现了一个反例模型，其中Adam爱某个女人而不是Eve。
```
>>>print mc.valuation
{'C1':'b',
'adam': 'a',
'eve': 'a',
'love': set([('a', 'b')]),
'man': set([('a',)]),
'woman': set([('a',), ('b',)])}
```
此外，我们并没有指定man和woman表示不相交的集合，因此，模型生成器让它们相互重叠。

因此，让我们添加一个新的假设，使man和woman不相交。模型生成器仍然产生一个反例模型，但这次更符合我们直觉的有关情况：
```
>>>a7 = lp.parse('all x.(man(x) -> -woman(x))')
>>>g= lp.parse('love(adam,eve)')
>>>mc= nltk.MaceCommand(g, assumptions=[a4, a5, a6, a7])
>>>mc.build_model()
True
>>>print mc.valuation
{'C1':'c',
'adam': 'a',
'eve': 'b',
'love': set([('a', 'c')]),
'man': set([('a',)]),
'woman': set([('b',), ('c',)])}
```
我们的假设中没有说`Eve`是论域中唯一的女性，所以反例模型其实是可以接受的。如果想排除
这种可能性，我们将不得不添加进一步的假设，如`exists y. allx.(woman(x) ->(x =y))`，
以确保模型中只有一个女性。



