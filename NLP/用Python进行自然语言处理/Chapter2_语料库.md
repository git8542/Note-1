####目录
- 1.基本语料库函数
- 2.载入自定义语料库
- 3.条件频率分布
	* 条件频率分布
	* 绘制分布图和分布表
	* 条件频率分布常见函数
- 4.WordNet

#1.基本语料库函数
<table>
<tr>
<td>fileids()</td>
<td>语料库中的文件</td>
</tr>
<tr>
<td>fileids([categories])</td>
<td>这些分类对应的语料库中的文件</td>
</tr>

<tr>
<td>categories()</td>
<td>语料库中的分类</td>
</tr>
<tr>
<td>categories([fileids])</td>
<td>这些文件对应的语料库中的分类</td>
</tr>

<tr>
<td>raw()</td>
<td>语料库中的原始内容</td>
</tr>
<tr>
<td>raw(fileids=[f1,f2,f3])</td>
<td>指定文件的原始内容</td>
</tr>
<tr>
<td>raw(categories=[c1,c2])</td>
<td>指定分类的原始内容</td>
</tr>

<tr>
<td>words()</td>
<td>整个语料库中的词汇</td>
</tr>
<tr>
<td>words(fileids=[f1,f2,f3])</td>
<td>指定文件中的词汇</td>
</tr>
<tr>
<td>words(categories=[c1,c2])</td>
<td>指定分类中的词汇</td>
</tr>

<tr>
<td>sents()</td>
<td>整个语料库中的句子</td>
</tr>
<tr>
<td>sents(fileids=[f1,f2,f3])</td>
<td>指定文件中的句子</td>
</tr>
<tr>
<td>sents(categories=[c1,c2])</td>
<td>指定分类中的句子</td>
</tr>

<tr>
<td>abspath(fileid)</td>
<td>指定文件在磁盘上的位置</td>
</tr>
<tr>
<td>encoding(fileid)</td>
<td>文件的编码（如果知道的话）</td>
</tr>
<tr>
<td>open(fileid)</td>
<td>打开指定语料库文件的文件流</td>
</tr>
<tr>
<td>root()</td>
<td>到本地安装的语料库根目录的路径</td>
</tr>
</table>

# 2.载入自定义语料库
使用 **PlaintextCorpusReader(corpus_ root, file_list/pattern)** 

*代码示例*：

	from nltk.corpus import PlaintextCorpusReader
	corpus_root = r'/user/share/dict'
	wordlists = PlaintextCorpusReader(corpus_root, '.*')
	print wordlists.fileids()

# 3.条件频率分布

(1) **条件频率分布**

这里的条件指的是文本的类别。当文本被分为几类，我们可以计算出每个类别独立的频率分布。我们不仅仅只是想知道整个语料库中词的频率分布，我们还可能想知道语料库中言情小说中（假设我们的语料库包含言情小说、玄幻小说等题材分类）词的频率分布。

ConditionalFreqDist接收一个以（condition_word,word）为元素的list。

	In[2]: from nltk.corpus import brown
	In[3]: import nltk
	Backend TkAgg is interactive backend. Turning interactive mode on.
	In[6]: cfd = nltk.ConditionalFreqDist((genre, word) for genre in brown.categories() for word in brown.words(categories=genre))
	In[7]: cfd
	Out[7]: <ConditionalFreqDist with 15 conditions>
	In[9]: cfd.conditions
	Out[9]: <bound method ConditionalFreqDist.conditions of <ConditionalFreqDist with 15 conditions>>
	In[10]: cfd.conditions()
	Out[10]: 
	[u'mystery',
	 u'belles_lettres',
	 u'humor',
	 u'government',
	 u'fiction',
	 u'reviews',
	 u'religion',
	 u'romance',
	 u'science_fiction',
	 u'adventure',
	 u'editorial',
	 u'hobbies',
	 u'lore',
	 u'news',
	 u'learned']
	In[11]: cfd[u'news']
	Out[11]: FreqDist({u'the': 5580, u',': 5188, u'.': 4030, u'of': 2849, u'and': 2146, u'to': 2116, u'a': 1993, u'in': 1893, u'for': 943, u'The': 806, ...})

（2）**绘制分布图和分布表**

ConditionalFreqDist为绘图和制表提供了一些方法，常用的有plot(),tabulate().

它们都可以接收conditions参数(list)和samples参数（list）。


	接（1）
	In[16]: cfd.tabulate(conditions=['fiction'], samples=['I', 'love', 'you'])
           I love  you 
	fiction  511   16  236 
	In[17]: cfd.plot(conditions=['fiction'], samples=['I', 'love', 'you'])

![](http://i.imgur.com/zNIF3s0.png)

（3）**条件频率分布常见函数**
<table>
<tr>
<td>cfdist=ConditionalFreqDist(pairs)</td>
<td>从配对链表中创建条件概率分布</td>
</tr>

<tr>
<td>cfdist.conditions()</td>
<td>将条件按字母排序</td>
</tr>

<tr>
<td>cfdist[condition]</td>
<td>此条件下的频率分布</td>
</tr>

<tr>
<td>cfdist[condition][sample]</td>
<td>此条件下给定样本的频率</td>
</tr>

<tr>
<td>cfdist.tabulate(samples, conditions)</td>
<td>指定样本和条件下的制表</td>
</tr>

<tr>
<td>cfdist.plot(samples, conditions)</td>
<td>指定样本和条件下的绘图</td>
</tr>

<tr>
<td>cfdist1 < cfdist2</td>
<td>测试样本在cfdist1中出现的频率是否小于在cfdist2中出现的次数</td>
</tr>
</table>

# 4.WordNet
[https://zh.wikipedia.org/wiki/WordNet](https://zh.wikipedia.org/wiki/WordNet)