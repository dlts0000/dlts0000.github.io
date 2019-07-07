# LSTM+CRF实现命名实体识别
## 1. LSTM
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
循环神经网络(RNNs)以一个序列$(x_1, x_2, ..., x_n)$为输入，返回另一个序列$(h_1, h_2, ...,h_4)$,输出序列代表了输入序列某一步中的某些信息。虽然从理论上讲，RNNs可以学习序列中的长依赖性，但在实际中却无法做到这一点，他们往往偏向于序列中最近的输入(梯度消失问题)。LSTMs通过引入记忆单元(memory-cell)来解决这个问题，并且证明可以解决长序列的依赖问题。lstm的解决方案是通过几个门来控制输入到记忆单元的比例和先前状态的遗忘比例。<br>
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
LSTM单元结构如下:
$$ 
\begin{align}
& i_t = σ(W_{xi}x_t + W_{hi}h_t−1 + W_{ci}c_{t−1} + b_i) \\
& c_t =(1−i_t)⊙c_{t−1}+
i_t ⊙ tanh(W_{xc}x_t + W_{hc}h_t−1 + b_c) \\
& ot = σ(W_{xo}x_t + W_{ho}h_t−1 + W_{co}c_t + b_o) \\
& h_t = o_t ⊙ tanh(c_t)
\end {align}
$$
其中σ 代表矩阵相乘，⊙ 是元素相乘。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
对于给定序列$(x_1, x_2, ..., x_n)$ 包含$n$个单词, 每一个单词代表一个$d$维向量. 对于每一个单词，LSTM计算句子的左向上下文表示$\overrightarrow{h_t}$；同样，产生一个右向上下文表示$\overleftarrow{h_t}$能够增加有用信息，这可以通过用第二个LSTM读取反向的输入序列来实现。通常称第一个为前向LSTM，第二个为后向LSTM，他们是两个具有不同参数的网络，合成双向LSTM。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
利用此模型，对于一个单词，将左右上下文连接起来的表示为单词的最终表示，$h_t = [\overrightarrow{h_t} ;\overleftarrow{h_t}]$. 这些表示能够有效的代表单词的上下文信息，这在许多标记应用中非常有用.
## 2. CRF标记模型
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
直接以$h_t$作为特征独立标记每一个输出$y_t$简单而有效，尽管在POS这类标记问题中有好的效果，但基于每一个输出作出独立分类判断在标记之相互依赖的时候就有局限性。NER就是这类问题，因为表征可解释的标签序列的“语法”强加了几个硬约束。(例如:  I-PER不能跟随B-LOC出现)。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
因此，我们不是独立对模型标记决策建模，而是通过条件随机场来对他们进行联合建模，对于一个句子，
$$X = (x_1,x_2,...,x_n),$$
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
考虑$P$是双向LSTM的得分输出，形如$n \times k$,其中$k$是标记总数，$P_{ij}$是句子中第$i$个单词对应于第$j$个标记的得分。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
对于一个句子的预测输出
$$y = (y_1,y_2,...,y_n),$$

我们定义它的得分为
$$
s(X,y) = \sum_{i=0}^nA_{yi,yi+1} + \sum_{i=1}^nP_{i,yi}
$$

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
其中A是转移矩阵，$A_{i,j}$表示从标记$i$转移到标记$j$的概率(但有些算法实现是相反的)，$y_0$和$y_n$是句子的起始标记和结束标记，我们将起始标记和结束标记加入句子的标记集合，因此A是大小为 $k+2$的方阵。

A softmax over all possible tag sequences yields a probability for the sequence y:

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
通过对所有可能的标记序列执行softmax函数，形成序列$y$的概率分布。
$$
p(y|X) = \frac{e^{s(X,y)}}{\sum_{\tilde{y}􏰂∈Y_X}e^{s(X,\tilde{y}􏰂)}}
$$

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
在训练过程中，我们最大化正确标记序列的对数概率:
$$
log(p(y|X))=s(X,y)−log\left(\sum_{\tilde{y}∈Y_X}e^{s(X,\tilde{y}􏰂)}\right)\\
= s(X, y) − log\,add_{\tilde{y}􏰂∈Y_X} s(X, y􏰂),
$$

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
其中$Y_X$表示所有可能的标记序列对于输入X，从公式可知，我们鼓励系统产生有效的输出标签序列。解码时，我们通过获取最大得分的序列来预测输出，其中:
$$
y^∗ =argmax_{\tilde{y}∈Y_X}s(X,\tilde{y})
$$

## 3.参考
1. [Neural Architectures for Named Entity Recognition](https://arxiv.org/abs/1603.01360)
2. [Cmd Markdown 公式指导手册](https://www.zybuluo.com/codeep/note/163962)
3. [理解LSTM网络](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)



