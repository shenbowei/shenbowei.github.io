---
layout: post
title: 读书笔记：数学之美-隐含马尔可夫模型
category: 读书
author: shenbowei
tags: 数学之美
---

**keyword**：`隐含马尔可夫模型`

本章是第三章的进一步拓展，推荐结合[数学之美-统计语言模型](https://shenbowei.github.io/2016/10/22/beauty-of-math-chapter3.html "跳转")一起看。

## 通信模型

下图表示一个典型的通信系统：

<center>
<img src="/public/img/beauty_of_math/5.1.png" width="70%" height="70%"  alt="通信模型" title="通信模型"/>
<br />
通信模型
</center>

上图是：

$S_1,S_2,S_3, \cdots$表示信息源发送的信号。

$O_1,O_2,O_3, \cdots$表示接收器接收到的信号。

通信中的解码就是根据接收到的信号$O_1,O_2,O_3, \cdots$还原出发送信号$S_1,S_2,S_3, \cdots$。

类比到自然语言处理，例如语音识别：

```

信息源：说话者的大脑

信道：喉咙，空气

接收器：听众的耳朵

信号：听到的声音

===》语音识别：根据声学信号来推测说话者的意思。如果接收端是一台计算机，那么就要做语音的自动识别。

```

### 如何根据观测信号推测信号源发送的信息？

问题： $O_1,O_2,O_3, \cdots$ ===》 $S_1,S_2,S_3, \cdots$ ？

从所有源信息中找到最有可能产生出观测信息的那一个。概率论的语言描述，已知$O_1,O_2,O_3, \cdots$，令

$P(S_1,S_2,S_3, \cdots \| O_1,O_2,O_3, \cdots)$达到最大值的那个信息串$S_1,S_2,S_3, \cdots$。即

$$
S_1,S_2,S_3, \cdots = \mathop{\arg\max}_{all S_1,S_2,S_3, \cdots}P(S_1,S_2,S_3, \cdots | O_1,O_2,O_3, \cdots)
$$

上面的概率不容易计算，利用贝叶斯公式转换：

$$
\frac{P(O_1,O_2,O_3, \cdots | S_1,S_2,S_3, \cdots) \cdot P(S_1,S_2,S_3, \cdots)}{P(O_1,O_2,O_3, \cdots)}
$$

其中：

$P(O_1,O_2,O_3, \cdots \| S_1,S_2,S_3, \cdots)$:表示信息$S_1,S_2,S_3, \cdots$在传输后变为$O_1,O_2,O_3, \cdots$的概率。

$P(S_1,S_2,S_3, \cdots)$:表示$S_1,S_2,S_3, \cdots$本身是一个在接收端合乎情理的信号（可以参考[数学之美-统计语言模型](https://shenbowei.github.io/2016/10/22/beauty-of-math-chapter3.html "跳转")）。

$P(O_1,O_2,O_3, \cdots)$：表示发送端产生$O_1,O_2,O_3, \cdots$的可能性（一旦信息$O_1,O_2,O_3, \cdots$产生了，它就不会改变了，这时$P(O_1,O_2,O_3, \dots)$就是一个可以忽略的常数。**表示没有太理解**）。

因此只需计算：

$$
P(O_1,O_2,O_3, \cdots | S_1,S_2,S_3, \cdots) \cdot P(S_1,S_2,S_3, \cdots)   (5.3)
$$

这个公式可以用**隐含马尔可夫模型（HMM）**来估计。

## 隐含马尔可夫模型

下图表示一个离散的马尔科夫过程：

<center>
<img src="/public/img/beauty_of_math/5.3.png" width="60%" height="60%"  alt="马尔可夫链" title="马尔可夫链"/>
<br />
马尔可夫链
</center>

四个圈代表四个状态，每条边表示一个可能的状态转移，边上的权值为转移概率。例如：$P(S_{t+1} = m_3 \| S_t = m_2) = 0.6$。

在把马尔可夫链上随机选择一个状态作为初始，按照链上的状态转换运行一段时间$T$后会得到一系列状态：$S_1,S_2S_3,\cdots S_T$。
从这个这个序列我们可以得到：

- #$(m_i)$:状态$m_i$出现的次数。

- #$(m_i,m_j)$:从状态$m_i$转换到$m_j$的次数。

从而计算处#$(m_i,m_j)/$#$(m_i)$，即从状态$m_i$转换到$m_j$的转移概率。

下面进入正题：**隐含马尔可夫模型**。
隐含马尔可夫模型是上边马尔可夫模型的拓展：

- 任意时刻$t$的状态$S_t$是不可见的。-----因此无法通过观察状态序列$S_1,S_2S_3,\cdots S_T$来推算转移概率等参数。

- 在每个时刻$t$会输出一个符合$O_t$,而且$O_t$仅跟$S_t$相关。-----称为独立输出假设。

隐含马尔可夫模型如下图所示，状态序列$S_1,S_2S_3,\cdots S_T$为典型的马尔可夫链：

<center>
<img src="/public/img/beauty_of_math/5.4.png" width="60%" height="60%"  alt="隐含马尔可夫模型" title="隐含马尔可夫模型"/>
<br />
隐含马尔可夫模型
</center>

基于**马尔可夫假设**和**独立输出假设**，我们可以计算出某个特定的状态序列$S_1,S_2S_3,\cdots S_T$产生出输出序列符号$O_1,O_2,O_3, \cdots$的概率：

$$
P(S_1,S_2S_3,\cdots ,O_1,O_2,O_3, \cdots) = \prod_{t}P(S_t|S_{t-1}) \cdot P(O_t|S_t)    (5.4)
$$

公式(5.4)和公式(5.3)非常相似，现在把**马尔可夫假设**和**独立输出假设**用在通信解码问题(5.3):

$$
P(O_1,O_2,O_3, \cdots | S_1,S_2,S_3, \cdots) = \prod_{t}P(O_t|S_t)
$$

$$
P(S_1,S_2,S_3, \cdots) =  \prod_{t}P(S_t|S_{t-1})
$$

上面的两个等式代入(5.3)整好可以得到(5.4),这样通信解码问题即可使用隐含马尔可夫模型来解决。

> 至于如何找到上面式子的最大值，进而找出要识别的句子$S_1,S_2S_3,\cdots S_T$，可以利用维特比算法(Viterbi Algorithm)。本书26章会介绍……(好遥远^_^\|\|)

## 隐含马尔可夫模型的训练

隐含马尔可夫模型的三个基本问题：

1. 给定一个模型，如何计算某个特定的输出序列的概率；（这个问题相对简单，对应的算法是Forward-Backward算法）

2. 给定一个模型和某个特定的输出序列，如何找到最可能产生这个输出的状态序列；（可以用著名的维特比算法解决）

3. 给定足够量的观测数据，如何估计隐含马尔可夫模型的参数；（即本节讨论的模型训练问题）

隐含马尔可夫模型的参数包括：

- $P(S_t\|S_{t-1})$：从前一个状态$S_{t-1}$进入当前状态$S_{t}$的**转移概率**

- $P(O_t\|S_t)$：每个状态$S_{t}$产生相应输出符号$O_{t}$的**生成概率**

计算或者估计这些参数的过程就是模型的训练。

通过条件概率，可知：

$$
P(O_t|S_t) = \frac{P(O_t,S_t)}{P(S_t)}    (5.6)\\
P(S_t|S_{t-1}) = \frac{P(S_{t-1},S_t)}{P(S_{t-1})}    (5.7)
$$

有如下两种训练方法：

### 有监督的训练方法（Supervised Training）

训练需要有足够多人工标记（Human Annotated）的数据。
从而统计处：

 - #$(S_t)$：经过状态$S_t$的次数；

 - #$(O_t,S_t)$：每次经过这个状态$S_t$时分别产生的输出$O_t$是什么，且分别产生的次数。
 
那么(5.6)可得:
 
$$
P(O_t|S_t) \approx \frac{\# (O_t,S_t)}{\# (S_t)}
$$ 

对于(5.7)的转移概率，和[数学之美-统计语言模型](https://shenbowei.github.io/2016/10/22/beauty-of-math-chapter3.html "跳转")的条件概率完全相同：

$$
P(S_t|S_{t-1}) \approx \frac{\# (S_{t-1},S_t)}{\# (S_{t-1})}
$$ 

**缺点：**需要大量的人工标记数据，遗憾的是很多应用都不可能做到这件事。即使可以需要的人工成本也非常高。

### 无监督的训练方法（Unsupervised Training）

仅仅通过大量观测到的信号$O_1,O_2,O_3, \cdots$就能推算模型参数$P(O_t\|S_t)$和$P(S_t\|S_{t-1})$的方法。主要使用的是**鲍姆-韦尔奇算法（Baum-Welch Algorithm）**。

> 两个不同的马尔可夫模型可以产生相同的输出信号，因此，仅从观测信号倒推模型可以得到多个合适的模型。但是总会有一个模型$M_{\theta_2}$比另一个模型$M_{\theta_1}$更有可能产生观测到的输出，
> 其中$\theta_2$和$\theta_1$是HMM的参数。**鲍姆-韦尔奇算法**就是用来寻找这个最可能的模型$M_\hat{\theta}$。

**鲍姆-韦尔奇算法**的思想：

- 首先找到一组能够产生输出序列$O$的模型参数，作为我们的初始模型$M_{\theta_0}$;（一定存在，因为转移概率和输出概率为均匀分布时，模型可以产生任何输出。）

- 在此基础上迭代优化。假设解决了前边提到的第一个问题和第二个问题，那么可以算出这个模型产生$O$的概率$P(O|M_{\theta_0})$以及找到产生$O$的所有可能路径、这些路径的概率。
 因此我们就得到了"标注的训练数据"，根据公式(5.6)和(5.7)计算新的模型参数$\theta_1$。可以证明：$P(O\|M_{\theta_1})>P(O\|M_{\theta_0})$。
 
- 输出上述迭代过程。

**缺点：**鲍姆-韦尔奇算法不断迭代以得到新的参数，使得输出的概率（目标函数）达到最大化，这个过程称为期望值最大化（Expectation-Maximization），简称EM过程。
EM过程保证算法一定能收敛到一个局部最优点，很遗憾它一般不能保证找到全局最优点（如果目标函数为凸函数，就可以找到唯一的最优点）。因此，这种无监督的鲍姆-韦尔奇算法训练得到的模型比有监督的训练得到的模型效果略差。






