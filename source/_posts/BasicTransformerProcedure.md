---
title: Basic Transformer Procedure(building...)
date: 2025-04-30 19:29:00
tags:
---
**Transformer文本处理流程浅析**
## Word Embedding词嵌入
将每一个词转化为一个n维向量（GPT-3有12288维，50257个词，组成第一个参数矩阵）
* 语义越接近，向量夹角越小。
* 具有一定的语义关联推理能力。
![wordEmbedding.png](/images/wordEmbedding.png)
如向量cats-cat可以近似得到向量空间中代表复数的向量
* 也可以用向量之间的点积衡量向量之间的对其程度。
如将plu复数向量与复数词点积的结果大于单数词，而且plur向量与表示自然数的向量的点积是递增的
* 此时的向量没有与上下问结合，可能携带多重含义。
* 还要将该词的位置（第几词）编码到向量中
## Attention
#### 单头注意力
#### 多头注意力
## MLP多层感知
## Unembedding解嵌入
结构类似Embedding矩阵的转置，每一行对应一个词，每一列
building.....
