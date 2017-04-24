#### 目录
- 1.创建一个Text对象
- 2.词干提取
- 3.词形归并
- 4.分词


# 1.创建一个Text对象
nltk.Text 构造器接收一个分词后的token，以此来创建Text对象。所以在创建Text对象前请分词。
```python

# -*- coding: utf-8 -*-
import nltk

raw = u"""He gave a sudden start; another thought, that he had had yesterday,
	slipped back into his mind. But he did not start at the thought
	recurring to him, for he knew, he had _felt beforehand_, that it must
	come back, he was expecting it; besides it was not only yesterday's
	thought. """
tokens = nltk.word_tokenize(raw)
text = nltk.Text(tokens)
print text.concordance(u'was')  # 打印was出现的所有上下文语境

```
# 2.词干提取
nltk提供了两个词干提取器，Porter和Lancaster。

*用法*

```python
word = u'lying'
porter = nltk.PorterStemmer()
landcaster = nltk.LancasterStemmer()
print porter.stem(word)
print landcaster.stem(word)
```
# 3.词形归并

WordNet词形归并器删除词缀产生的词都是在词典中有的词。它比词干提取多了一个额外的检查过程。

*用法*
```python	
word = u'women'
wnl = nltk.WordNetLemmatizer()
print wnl.lemmatize(word)
```
# 4.分词

对于词的边界有明显分割的语言（比如英文），使用简单的正则表达式根据空格即可分词。

对于词的边界没有明显分割的语言（比如中文），本书提出了构造目标函数，最优化目标的分词方法。（p118）



