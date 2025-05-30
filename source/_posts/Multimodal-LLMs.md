---
title: Multimodal LLM
date: 2025-05-29 11:07:44
tags:
---
**MLLM实现原理**
有两种实现MLLM的方法：
(https://magazine.sebastianraschka.com/p/understanding-multimodal-llms)
* **统一嵌入解码器（统一编码器）Unified Embedding Decoder Architecture approach：**
  更像未经修改的纯解码器LLM，如GPT-2或者Llama3.2。图像被转换成与输入文本相同大小的token。连接后由LLM一同处理。
* **跨模态注意力Cross-modality Attention Architecture approach**
![Mllmapproches](/images/Mllmapproches.png)
## 图像编码器：从VIT到CLIP
#### VIT(vision transformer)
from“AN IMAGE IS WORTH 16X16 WORDS:TRANSFORMERS FOR IMAGE RECOGNITION AT SCALE”,Google
![MLLM_VITstruct](/images/MLLM_VITstruct.png)
* 证明了transformer的通用性
* 原本是图像分类，使用transformer可以不改动处理视觉分类任务
* 大规模数据集上强于卷积神经网络且计算效率更有优势
流程：
* 图像切片成batch size，如16*16，相当于text的分词
* 展平成一维向量后进入线性投影层，相当于embedding，转换成为更高维向量
* 位置编码，使用可学习的位置编码向量（类似bert），直接加在token向量上
* 由于是做图像分类（信息提取）而非内容生成，transformer encoder使用类似bert的L层encoder架构
* 最后送入一个MLP头进行分类
#### Clip(Contrastive Language-Image Pre-training)
from“Learning Transferable Visual Models From Natural Language Supervision”,OpenAI
![MLLM_clip](/images/MLLM_clip.png)
* 在VIT有限分类的基础上，不用对图片标注就能训练，能分类无限多类
* 采用双编码器架构，由图像编码器和文本编码器组成。图像编码器用于将图像映射到特征空间，**可选择VIT（最佳，要去掉MLP分类头）**，ResNet等不同架构，进行模糊池化、将全局平均池化替换为注意力池化等调整；文本编码器用于将文本映射到相同的特征空间，采用12层、512的隐层节点数以及8个头的 Transformer 。
* 训练时类似做连线题。给定一批N个（图像，文本）对，CLIP训练目的是预测这批数据中N×N种可能的（图像，文本）配对中哪些是正确对应的。
* 通过计算图像编码器和文本编码器的向量输出的余弦相似度，选择最相似的类别，计算交叉熵loss反向传播，同时更新图像和文本编码器参数。
* **零样本任务迁移zero-shot transfer**：
  可以不做微调，直接用预训练好的模型实现通用的分类。但**这个还不是MLLM的给定一个简单query做任何任务**，而是必须在prompt中给定所有可能的类别。比如如果要MINIST手写数字识别，就必须在prompt中写“1，2，3，4，5，6，7，8，9，0”十个类别，送入text encoder，生成十个向量，要识别的图片送入image encoder，生成一个向量，比较他和十个向量之间的余弦相似读来分类。

**在Unified Embedding Decoder MLLM中，image encoder实际上取的就是Clip的图像编码器部分**

## 统一嵌入解码器架构
具体需要分析这个能将图像与文本**在Unified Embedding Decoder MLLM中，image encoder实际上取的就是Clip的图像编码器部分**对齐的image encoder：
![Mllm_encoder](/images/Mllm_encoder.png)
这里实际上是VIT的流程：
- 先将图像分割为标准大小batch，比如16*16的图片块。
- 随后由线性层linear projection统一维度，比如16*16的图片块展平后的256个像素经全连接层投影变为768维向量。
- 随后送入transformer encoder，实际应用中没有VIT原始的MLP Classcification Head，输出向量还要经过一层线性Projection层，映射回输入text token的维度。


## 跨模态注意力Cross-modality Attention Architecture approach
![MLLM_cross_attention](/images/MLLM_cross_attention.png)
类似与transformer原本的翻译器结构，将transformer原本的text encoder替换为Clip图像编码器（仅有残差连接的自注意力层和MLP层，且自注意力层无掩码），送入原本解码器内的编码器-解码器注意力层，这个注意力层于是就被替换为图像-文本交叉注意力层。
![MLLM_CrossAttention](/images/MLLM_CrossAttention.png)
## Llama3.2 Vision
https://www.bilibili.com/video/BV1QrBEY3Eax/?share_source=copy_web&vd_source=275d46b9a03d7ce577d10c1f2bdb1206
* 图像编码器：
  -  先将图片分为最多四个Tile（Tile之间有八种不同组合方式），每个都是$3*560*560$（放不下通过图像算法缩放），用掩码如[1,1,0,0]来表示哪些Tile内有图片内容。位置编码通过加Tail Pos Embedding实现。
  -  每个Tile划分为$40*40$个$14*14*3$大小的patch，每个path展平后经过线性投影层映射为1280维向量，总共拍成一个序列就是$6400*1280$的张量。
  -  transformer encoding共40层，前面32层关注局部特征，后面八层在注意力和前项传播层加了**tanh门控单元**，使模型更关注全局特征，最后输出的全局特征与3,7,15,23,30层的全局特征合并，输出$6400*7680$的张量。
* 文本编码器：
  - 在输入字典内，新增<|image|>的token，作为占位符，表示图片所在位置。
  - 编码器使用Llama3.1预训练编码器，训练过程中保持冻结。
* 解码器：
  - 在3,8,13,18,23,28,33,38层使用交叉注意力，query来自上一层输出，key和value来自图像解码器的固定输出。
  - **通过注意力掩码，确保在跨注意力层中，每个文本token只能看到前面的一张图片。但如果图片与图片之间没有text，则文本token可以看到前面的连续多张图片。这是因为图片后的文本token可以通过注意力学习到图片特征，而保留之前的多张图片会丢失image与text之间的对应关系。**