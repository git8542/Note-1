**目录**

1. 分词任务
2. 词性标注任务

# 1. 分词任务
**流程**：

1. 获取标注数据集（观测序列是输入句子，状态序列是句子对应的BMES标记）
2. 利用极大似然估计方式估计HMM模型三个参数矩阵——初始状态概率矩阵、状态转移概率矩阵、观测发射概率矩阵。（有监督的HMM参数估计算法详见《统计学习方法》10.3.1节）
3. 最好对参数矩阵中的每个概率值取对数，原因是可以将概率相乘计算变成对数相加，这在预测时很有用。概率`0`值的对数（负无穷）使用`3.14e+100`表示。
4. 用维特比算法（或者直接使用[jieba.finalseg.cut()](https://github.com/fxsjy/jieba/blob/master/jieba/finalseg/__init__.py)方法）使用上面的参数预测分词。
5. 使用http://sighan.cs.uchicago.edu/bakeoff2005/ 提供的score脚本及测试数据测试模型，获得模型相关评测指标。（使用方式都在下载的文件的README内）

*Code Example*

```python
MIN_FLOAT = -3.14e+100

# 观测发射矩阵
pro_emit = {
    u'B': {},
    u'M': {},
    u'E': {},
    u'S': {}
}
# 初始状态矩阵
pro_start = {
    u'B': 0,
    u'M': 0,
    u'E': 0,
    u'S': 0
}
# 状态转移矩阵
pro_trans = {
    u'B': {u'B': 0, u'M': 0, u'E': 0, u'S': 0},
    u'M': {u'B': 0, u'M': 0, u'E': 0, u'S': 0},
    u'E': {u'B': 0, u'M': 0, u'E': 0, u'S': 0},
    u'S': {u'B': 0, u'M': 0, u'E': 0, u'S': 0}
}

def estim_param(file_path):
    """
    使用数据集估计HMM参数
    （文件格式：每一行为一个句子；每个句子被空格分成多个单词）
    :param file_path: 分好词的文件路径
    :return: 
    """
    """
    统计三个参数矩阵中每个元素的频数
    """
    for idx, line in enumerate(codecs.open(file_path, encoding='utf-8')):
        new_line = line.strip()
        observe_seq = u''  # 观测序列
        state_seq = u''  # 状态序列
        for word in new_line.split():
            # 对每个word生成观测序列
            observe_seq += word
            # 对每个word生成状态序列
            word_length = len(word)
            if word_length == 1:
                state_seq += u'S'
            elif word_length > 1:
                state_seq += u'B'
                state_seq += u'M' * len(word[1:-1])
                state_seq += u'E'
        # 获取序列长度
        seq_length = len(state_seq)
        if seq_length > 0:
            # 统计获得状态转移矩阵（矩阵元素目前是频数）
            for i in xrange(seq_length - 1):
                pro_trans[state_seq[i]][state_seq[i + 1]] += 1
            # 统计初始状态矩阵（矩阵元素目前是频数）
            pro_start[state_seq[19]] += 1  # 人民日报语料库第19个字符才为真实句子开始符（前面是新闻排列号）
            # 统计观测发射矩阵（矩阵元素目前是频数）
            for i in xrange(seq_length):
                if observe_seq[i] not in pro_emit[state_seq[i]]:
                    pro_emit[state_seq[i]][observe_seq[i]] = 1
                else:
                    pro_emit[state_seq[i]][observe_seq[i]] += 1
    """
    根据频数矩阵生成概率矩阵
    """
    # 状态转移概率矩阵
    for init_state in pro_trans:
        total = sum(pro_trans[init_state].values())
        for transfer_state in pro_trans[init_state]:
            pro_trans[init_state][transfer_state] /= float(total)
            pro_trans[init_state][transfer_state] = log(pro_trans[init_state][transfer_state])
    # 初始状态概率矩阵
    total = sum(pro_start.values())
    for state in pro_start:
        pro_start[state] /= float(total)
        pro_start[state] = log(pro_start[state])
    # 观测发射概率矩阵
    for state in pro_emit:
        total = sum(pro_emit[state].values())
        for observe in pro_emit[state]:
            pro_emit[state][observe] /= float(total)
            pro_emit[state][observe] = log(pro_emit[state][observe])
def log(x):
    """
    对x取log（增加了x是否为0检查）
    :param x: 
    :type x: float
    :return: 
    """
    return math.log(x) if x else MIN_FLOAT
```

# 2. 词性标注任务
**流程**：

1. 获取标注数据集。数据必须带分词及词性标注。（观测序列为输入句子，状态序列是`('B', 'n')`二元组格式，第一个元素是BMES标记，第二个元素是词性标记）
2. 利用极大似然估计方式估计HMM模型三个参数矩阵——初始状态概率矩阵、状态转移概率矩阵、观测发射概率矩阵。还需要获得观测状态表（对应于jieba的`char_state_tag`表，它表示每个观测所可能有的状态）
3. 最好对参数矩阵中的每个概率值取对数，原因是可以将概率相乘计算变成对数相加，这在预测时很有用。概率`0`值的对数（负无穷）使用`3.14e+100`表示。
4. 用维特比算法（或者直接使用[jieba.posseg.cut()](https://github.com/fxsjy/jieba/blob/master/jieba/posseg/__init__.py)方法）使用上面的参数预测分词。