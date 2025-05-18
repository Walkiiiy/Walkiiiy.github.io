---
title: GPT-style LLM structure
date: 2025-05-15 19:29:00
tags:
---
**基础Transfermer的纯解码器LLM浅析**
常见的Llama,GPT都是基于纯解码器。 
![LLM](/images/LLM.jpg)
## Tokenize
* 首先大小写转换，特殊字符处理，统一为Utf-8编码。
* 使用sub-word粒度子词分词，一般使用BPE算法
  (https://zh.d2l.ai/chapter_natural-language-processing-pretraining/subword-embedding.html),(https://zhuanlan.zhihu.com/p/424631681)，原理类似哈夫曼编码。预训练BPE能构建大小适中的语料库（即token库），应用时将输入拆分为字符（utf-8），再按照token频次对应合并。
  （Llama模型中文处理不好部分原因就是缺少中文字符的token预训练）
* 将处理好的token序列转为对应数值的序列。
## Word Embedding
**可看作token序列到词嵌入空间的索引查询**
将每一个token转化为一个n维向量（GPT-3有12288维，50257个词，组成第一个参数矩阵）
* 语义越接近，向量夹角越小，具有一定的语义关联推理能力。
![wordEmbedding](/images/wordEmbedding.png)
如向量cats-cat可以近似得到向量空间中代表复数的向量
* 也可以用向量之间的点积衡量向量之间的对其程度，如将plu复数向量与复数词点积的结果大于单数词，而且plur向量与表示自然数的向量的点积是递增的。此时的向量没有与上下问结合，可能携带多重含义。
* 向量集合在反响传播中也会被训练更新。
* 除此之外，还要将该词的位置（第几词）编码到向量中，Transformer原始的固定位置编码(https://blog.csdn.net/m0_37605642/article/details/132866365)
$$
\begin{align*}
PE(p, 2i) &= \sin\left(\frac{p}{10000^{2i/d}}\right) \\
PE(p, 2i+1) &= \cos\left(\frac{p}{10000^{2i/d}}\right)
\end{align*}
$$
  - $p$为第几个向量，$d$为向量维数，$2i$和$2i+1$为第几维。  
  - 利用三角函数周期性，低维到高维频率增加，$p$为整数使得前后向量之间的数值永不相同，且位置向量的内积会随着相对位置的递增而减小。
将位置编码向量与词义向量相加，得到输出。
## Attention
#### 自注意力
**根据不同向量之间的语义关联程度，处理输入序列内所有向量之间的相互影响。**
![K and Q in self-attention](/images/KandQinself-attention.png)
假设输入序列为 $ X = [x_1, x_2, \dots, x_n] $，每个 $ x_i $ 是 $ d_{\text{model}} $ 维向量（如词嵌入向量）。  
通过可学习的矩阵 $ W^Q, W^K, W^V $ 将输入向量从原特征空间映射到三个新的子空间：  
- **Query空间**：用于定义“查询”的视角（我可以被哪些词影响）。  
- **Key空间**：用于定义“被检索”的键（我可以影响哪些词）。  
- **Value空间**：用于存储实际参与加权求和的信息（具体的影响）。  

每个矩阵的参数量为 $ d_{\text{model}} \times d_k $，通常 $ d_k = d_{\text{model}} $，因此总参数量为 $ 3 \times d_{\text{model}}^2 $。
另外，为了确保解码时每个位置只能关注其左侧（过去）的位置，不能看到未来的信息，以模拟真实推理过程，需要进行注意力掩码，即让上图矩阵中的左下部分不参与运算。
##### （1）点积相似度（Dot-Product Similarity）
$$
\text{scores}_{i,j} = q_i \cdot k_j = \sum_{m=1}^{d_k} q_{i,m} k_{j,m}
$$  
**意义**：点积衡量两个向量的相似性，值越大表示向量夹角越小（方向越一致），表示embedding空间中的向量j对i产生影响的程度大小。  
**缺陷**：当 $ d_k $ 较大时，点积结果的方差会增大（如向量各维度独立且均值为0时，点积方差为 $ d_k \sigma^2 $），导致Softmax函数梯度消失。

##### （2）缩放点积相似度（Scaled Dot-Product）
$$
\text{scores}_{i,j} = \frac{q_i \cdot k_j}{\sqrt{d_k}}
$$  
假设 $ q_i $ 和 $ k_j $ 是均值为0、方差为1的独立随机变量，则点积的均值为0，方差为 $ d_k $。缩放后方差归一化为1，使Softmax函数处于梯度稳定区域。
##### （3）Softmax归一化得到注意力分数：  
$$
\text{Attention Scores} = \text{Softmax}\left( \frac{Q K^T}{\sqrt{d_k}} \right) \in \mathbb{R}^{n \times n}
$$  
##### （4）值矩阵映射
$$
Z = \text{Softmax}\left( \frac{X W^Q (X W^K)^T}{\sqrt{d_k}} \right) X W^V
$$
随后每一个输入与注意力值z相加，该步骤被称为残差连接。
#### 多头注意力
![multi-head-attention](/images/multi-head-attention.svg)
同一个词在不同的语境下会对其他词产生不同的影响，但单头注意力却只设置唯一的查询、键值矩阵。为了让模型理解更复杂的语境，让不同注意力头关注不同类型的句法和语义关系。
* GPT3有96个头，12288维向量由每个头的128维向量拼接而成。
* 以GPT-3为例，**输入序列的每个token的12288维表示会通过线性变换，拆分为96个128维的Q/K/V向量组，这些向量组通过多头注意力机制并行计算，最终将96个头输出的128维结果拼接后经过输出投影矩阵，重新组合为12288维的输出向量**。
## MLP多层感知（前馈神经网络层FNN）
自注意力层捕捉输入词之间产生的关系，FNN可以为输入补充原本上下文中没有，但是应该存放在模型中的知识。**比如输入只有“迈克尔-乔丹”，Attention可以知道“姓乔丹名麦克尔”，但是“是一名篮球运动员”只有经过FNN才能得到。**
![FNNStructure](/images/FNNStructure.png)
FFN由两个线性变换（全连接层）和一个非线性激活函数组成：
$$
\text{FFN}(x) = \text{max}(0, xW_1 + b_1)W_2 + b_2
$$
其中：
- $x \in \mathbb{R}^{d_{\text{model}}}$ 是输入向量（通常为注意力机制的输出）；
- $W_1 \in \mathbb{R}^{d_{\text{model}} \times d_{\text{ff}}}$ 和 $W_2 \in \mathbb{R}^{d_{\text{ff}} \times d_{\text{model}}}$ 是权重矩阵；
- $b_1 \in \mathbb{R}^{d_{\text{ff}}}$ 和 $b_2 \in \mathbb{R}^{d_{\text{model}}}$ 是偏置向量；
- $\text{max}(0,  xW_1 + b_1）$是**ReLU激活函数**，引入非线性。
-  中间维度 $d_{\text{ff}}$ 通常是模型维度 $d_{\text{model}}$ 的4倍（如GPT-3中 $d_{\text{model}}=12288$，$d_{\text{ff}}=49152$）。
  
  计算过程中**W矩阵的49152行每一个向量相当于一个神经元，与自注意力层输出的向量X点乘，可以理解为对该向量携带的知识库信息进行映射。X内的每一个参数都参与运算，因此构成全连接层。** 中间向量的值大于0则神经元Active，小于0会被Relu归0。
  ![W in FFn](/images/WinFFn.png)
-  经过$W_1$，中间向量的维度增加四倍，经过Relu后携带经过$W_2$后被压缩回原来的维度。
-  最后同样经过残差连接，即与输入相加。
- 参数分布：  
  FFN通常占总参数的约3/4（如在 $d_{\text{ff}}=4d_{\text{model}}$ 时，参数比例为 $\frac{4d_{\text{model}}^2 + 4d_{\text{model}}}{5d_{\text{model}}^2 + 5d_{\text{model}}} \approx 75\%$）。




## Unembedding解嵌入
在GPT-3中，隐藏层经过**96层的Attention和MLP**，进入解嵌入。
* 将12288维的隐藏层向量$h$映射为50257个token的概率分布，**因此需要一个线性层，将12288维向量转为对应的logits向量：**
$$
Logits=h⋅W_{lm}​+b
$$
大部分模型的$W_{lm}$与嵌入层的$W_{emb}$互为转置。
* 随后进行softmax归一化，即可对应预测下一token的概率的向量。
