---
date: 2019-01-19
title: "Know LSTM"
tags:
    - Machine Learning
    - LSTM
    - note
categories:
    - Machine Learning
comment: true
mathjax: true
---

[生成句子CNN](https://tensorflow.google.cn/tutorials/sequences/text_generation)

[预测单词LSTM](https://tensorflow.google.cn/tutorials/sequences/recurrent#lstm)



## 背景知识

### RNN

循环神经网络的隐藏层中，输出不仅取决于当前的输入还取决于上一时间隐藏层的输出结果。

可以看到RNN包含历史信息（A recurrent neural network can be thought of as multiple copies of the same network, each passing a message to a successor.）

![An unrolled recurrent neural network.](https://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-unrolled.png)

RNN可以将历史信息关联到当前的任务中，比如根据视频的历史帧赋予对当前帧的理解（One of the appeals of RNNs is the idea that they might be able to connect previous information to the present task, such as using previous video frames might inform the understanding of the present frame.）

![p1](https://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-shorttermdepdencies.png)

如果相关信息和所需位置的差距很小，RNN就可以学习使用过去的信息（In such cases, where the gap between the relevant information and the place that it’s needed is small, RNNs can learn to use the past information.）

随着预测信息和相关信息间的**间隔增大**，RNN逐渐失去对较远数据的关联能力（context from further back），因而无法预测长序列。（vanishing gradient problem）

理论上RNN是可以处理这种long-term dependencies的，但这确实很难（[Hochreiter (1991)](http://people.idsia.ch/~juergen/SeppHochreiter1991ThesisAdvisorSchmidhuber.pdf) and [Bengio, et al. (1994)](http://www-dsi.ing.unifi.it/~paolo/ps/tnn-94-gradient.pdf)）。而LSTM没有这种问题。

### LSTM

https://colah.github.io/posts/2015-08-Understanding-LSTMs/

LSTM旨在避免long-term dependency 问题，记住长时间的信息是它的默认行为，所以它当然可以进行长序列的学习预测（ [Hochreiter & Schmidhuber (1997)](http://www.bioinf.jku.at/publications/older/2604.pdf)）。

所有的神经网络都有链式的重复模块，普通的RNN隐藏层中只含有单一的tanh层。而LSTM有四个相互作用的层，其隐藏层使用了遗忘门、传入门、输出门几个相互作用的层。

![lstm](https://upload.wikimedia.org/wikipedia/commons/3/3b/The_LSTM_cell.png)

#### LSTM的基本结构

Cell state 像传送带一样使信息保持不变的向前流动，在此期间有较少的线性交互。LSTM并不直接的更改 cell state 信息，而是由 gate 来控制，gate 实际上就是由 sigmoid 和点乘操作组成的门。Sigmoid通过输出介于[0,1]之间实数表示信息通过的量。LSTM有三个这样的 gate 。

##### 详细的步骤

如果想要 cell state 遗忘某些信息，LSTM 中的“遗忘门”（forget gate layer）可以达到这种效果。它的输入是上一步隐藏层 $h_{t-1}$ 和当前输入值 $x_t$ ，为 cell state 中的每个值都输出一个[0,1]之间的值，表示忘记的程度。

如果想要将一些新的信息存入 cell state 中，首先，采用“传入门”（input gate layer）来决定被更新的信息，接着，tanh layer 创建一个新的候选值 $\tilde{C}_t$ ，最后组合这两个值来更新 state。

最后，先根据 Sigmoid 决定 cell state 的输出部分，然后和tanh的值相乘，得到输出结果。

#### 其它不同的派生

Peephole connections，gate layers 可以直接获得 cell state。

Coupled forget and input gates，不同于单独处理忘记的信息和传入的信息，忘记和传入的信息同时处理。在传入一些信息的时候才会忘记这些信息，在忘记一些旧的信息的时候才会传入新的信息。

Gated Recurrent Unit，将忘记和传入门结合为一个单独的“更新门”，融合了 cell state 和 hidden state。

## 样例代码

[tutorial](https://tensorflow.google.cn/tutorials/sequences/recurrent)

> 该模型的核心由一个 LSTM 单元组成，该单元一次处理一个字词，并计算句子中下一个字词的可能值的概率。该网络的内存状态初始化为零矢量，并在读取每个字词后进行更新。

可能的数据形式

```python
 t=0  t=1    t=2  t=3     t=4
[The, brown, fox, is,     quick]
[The, red,   fox, jumped, high]

words_in_dataset[0] = [The, The]
words_in_dataset[1] = [brown, red]
words_in_dataset[2] = [fox, fox]
words_in_dataset[3] = [is, jumped]
words_in_dataset[4] = [quick, high]
batch_size = 2, time_steps = 5
```

伪代码

```python
words_in_dataset = tf.placeholder(tf.float32, [time_steps, batch_size, num_features])
lstm = tf.contrib.rnn.BasicLSTMCell(lstm_size)
# Initial state of the LSTM memory.
state = lstm.zero_state(batch_size, dtype=tf.float32)
probabilities = []
loss = 0.0
for current_batch_of_words in words_in_dataset:
    # The value of state is updated after processing each batch of words.
    output, state = lstm(current_batch_of_words, state)

    # The LSTM output can be used to make next word predictions
    logits = tf.matmul(output, softmax_w) + softmax_b
    probabilities.append(tf.nn.softmax(logits))
    loss += loss_function(probabilities, target_words)
```

LSTM：一组链式的Cell，指定lstm_size

每个batch对应state向量中的一个值

对每个batch：output, state = lstm(current_batch, state) ，state参数是从前一次迭代中得到

从output获得概率

截断的反向传播算法：处理每批后回馈长度为 `num_steps` 的输入

stack LSTM：`tf.contrib.rnn.MultiRNNCell`

结果：（困惑度）

```
Epoch: 8 Train Perplexity: 50.858
Epoch: 8 Valid Perplexity: 127.851

===========================================
| config | epochs | train | valid  | test
===========================================
| small  | 13     | 37.99 | 121.39 | 115.91
| medium | 39     | 48.45 |  86.16 |  82.07
| large  | 55     | 37.87 |  82.62 |  78.29
```
