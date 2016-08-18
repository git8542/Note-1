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


    