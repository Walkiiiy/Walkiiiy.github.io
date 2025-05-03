---
title: ChatTS:Aligning TimeSeries With LLMs
date: 2025-05-03 11:08:30
tags:
---
**文献阅读：通过合成数据集，构建具有更强理解分析能力的时间序列模态大模型ChatTS**
**from"ChatTS: Aligning Time Series with LLMs via Synthetic Data for Enhanced Understanding and Reasoning"by Zhe Xie,Zeyan Li,Xiao He,Longlong Xu,Xidao Wen,Tieying Zhang,Jianjun Chen,Rui Shi,Dan Pei**
## 总览
* 该研究：
  * 改进了时间序列数据集生成策略Time Series Generator。
  * 拓展指令演化方法Evol-Instruct为TSEvol，生成高质量数据集问答内容，对模型微调。
  * 改进TimeLLM架构为ChatTS架构，拥有更强的数据分析通用性，拥有了多变量时间序列（MTS）相关性分析能力，并且拥有了回溯精确数据的能力。
![overview](/images/CharTS:overall.png)

## 背景与需求
#### 传统LLM的时间序列分析方法
![conventional](/images/ChatTS_conventional.png)
* text-based：直接词嵌入，丢失数值理解能力和精度，且数据量大时，对数据的全局理解能力弱。
* agent-based：需要借助第三方工具，缺乏通用性。
* VisionMLLM：精度差。
#### 需求
![ChatTS_example](/images/ChatTS_example.png)
* 要求模型具有：
**精确**、可靠、**全局理解能力强**的时间相关数据分析，具有**理解分析不同类型数据之间相关性**的能力。输入多组数据和问题，返回分析结果，且能**精确回溯输入的每一精确数据**。
* 要求训练数据：
足够精确，多种任务以增强通用性
## 训练数据合成
#### 属性选择器Attribute Selector
* 将时间序列的属性分为四类：趋势，周期性，噪声，局部波动（Trend, Periodicity, Noise, and Local Fluctuation）
* 定义一个属性全集All attributes set，包含四种趋势，7种周期性，3种噪声，19种局部波动。
* 定义单位集Metric Set，包含真实应用中的567种预定义单位。
* 使用GPT选择符合符合预定义场景、单位的数据属性（使其更符合真实的物理应用场景），形成属性子集。
* 属性采样器随机从属性子集种抽取属性，并从GPT给定的合理单位中指定单位，指定合理的边界值，整体放入属性池Attribute Pool。
![TSGenerator](/images/ChatTS_TSGenerator.png)
#### 时间序列生成器Time Series Generator
* 属性池Attribute Pool中记录着该条数据所有具体的数据属性，如:
```
Trend Attribute：
[increase] from 50.0, reach
75.0 at point 150, and
[decrease] to 25 at point 256

Periodicity Attribute：
[(composite) sine] wave, with
period of 45.5 and amplitude
of 11.3

Local Fluctuation Attribute:
[upward spike] with amplitude of
28 at point 103; [downward spike]

Noise Attribute:
with amplitude of 23 at point 189
[noisy], the overall std is 3.35
```
* 时间序列生成器根据属性池Attribute Pool中的每一条描述生成具体的时间序列数组（具体见代码）
  
**至此，我们可以生成带有详细属性描述的时间序列（属性描述来自属性池Attribute Pool）**
#### 时间序列演化Time Series Evol-Instruct（TSEvol）
要训练ChatTS，需要的不仅仅是时间序列TS及其属性描述，还有高质且多样的问答示例，特别是需要训练分析不同数据实例之间相关性的能力。

##### **PreWork:Evol-Instruct**
origin work:WizardLM: Empowering Large Language Models to Follow Complex Instructions

Evol-Instruct是一种使用GPT作为进化引擎的LLM复杂指令生成技术，原本用于生成复杂度和多样性优于人类的指令数据集，用于对模型进行调优训练。（也就是通过逐步重写初始指令来生成复杂度更高的指令，并用这些数据微调模型）。在2023年提出，研究者使用Evol-Instruct演化的指令集对LLaMa微调，复杂任务上表现超过ChatGPT。
![evol-instruct，从种子问题开始，蓝色为深度进化，红色为广度进化](/images/evol-instruct.png)

研究提出两种演化策略：
• 深度进化（In-depth Evolving）：包括5种提升指令的复杂度策略
    ◦ 示例（添加约束）：
      ◦ 原指令：  
        “写一个Python函数计算斐波那契数列。”
      ◦ 进化后：  
        “写一个Python函数计算斐波那契数列，要求使用递归实现，且输出结果以JSON格式返回，包含前10项的值。”

• 广度进化（In-breadth Evolving）：生成与原指令主题相关但更小众的新指令，增加数据多样性。
◦ 示例：  
    ◦ 原指令：“解释光合作用的过程。”  
    ◦ 进化后：“解释深海热液喷口生态系统中的化能合成作用，并与光合作用进行对比。”

每种进化策略都有模板，如：
```text
"将以下指令改写为更复杂的版本，需添加JSON格式的输出要求，并限制答案在100字以内：  
[原始指令]"
  ```
通过特定提示模板指导GPT生成问题，另外数据集的回答也是GPT生成。
##### TSEvol in this work
**TSEvol用于从属性池Attribute Pool中生成时间序列数据集的Q&A，对ChatTS进行调优。**
![TSEvol](/images/TSEvol.png)
* 在深度和广度基础上，加入两种新的演化策略：
  * ​​推理（Reasoning）​​：要求模型结合物理意义进行归因分析（如“若某指标突增后另一指标延迟下降，可能的系统故障原因是什么？”）。
  * ​情境（Situation）​​：将时间序列嵌入特定场景（如“假设这是某医院的ICU监护数据，分析心率与血氧饱和度的关联”），增强任务多样性。
* 使用LLM结合属性池构建相关性池Correlation pool，专门用于生成多变量关联的问题，如：网络流量在30秒同步增加5Mbps（与CPU峰值相关），对此深入分析。
## ChatTS
#### PreWork:TimeLLM解决时间序列数据与自然语言模态对齐问题
**origin work: Time-llm:Time series forecasting by reprogramming large language models**
![TimeLLM](/images/TimeLLM.png)
TimeLLM可以看作是本研究的基础，通过输入时间序列和文本描述，通过多模态大模型MLLM对数据序列进行**预测**。
**时间序列与文本对齐的关键：跨注意力机制**
**1. 跨注意力的输入与输出**
• 输入：

  • 时间序列Patch嵌入：输入时间序列经分块（Patching）和线性投影后得到维度为 $ P \times d_m $ 的向量 $ \hat{X}_P^{(i)} $，其中 $ P $ 为块数，$ d_m $ 为嵌入维度。

  • 文本原型：从预训练LLM的词嵌入 $ E \in \mathbb{R}^{V \times D} $ 中选择或学习一组精简的文本原型 $ E' \in \mathbb{R}^{V' \times D} $，其中 $ V' \ll V $，$ D $ 为LLM的隐藏层维度。

• 输出：重编程后的时间序列表示 $ O^{(i)} \in \mathbb{R}^{P \times D} $，可直接输入冻结的LLM。


---

**2. 跨注意力的计算步骤**
1. 多头投影：
   • 对每个注意力头 $ k $，分别将时间序列Patch和文本原型投影到查询（Query）、键（Key）、值（Value）空间：

     $$
     Q_k^{(i)} = \hat{X}_P^{(i)} W_k^Q, \quad K_k^{(i)} = E' W_k^K, \quad V_k^{(i)} = E' W_k^V
     $$
     其中 $ W_k^Q \in \mathbb{R}^{d_m \times d} $, $ W_k^K, W_k^V \in \mathbb{R}^{D \times d} $，$ d = \lfloor \frac{d_m}{K} \rfloor $，$ K $ 为头数。

2. 注意力权重计算：
   • 计算每个时间序列Patch对文本原型的注意力权重：

     $$
     \text{Attention}(Q_k^{(i)}, K_k^{(i)}, V_k^{(i)}) = \text{Softmax}\left(\frac{Q_k^{(i)} K_k^{(i)\top}}{\sqrt{d_k}}\right) V_k^{(i)}
     $$
     通过缩放点积注意力（Scaled Dot-Product Attention）分配权重，关注最相关的文本原型。

3. 多头聚合与线性投影：
   • 拼接所有头的输出 $ Z_k^{(i)} \in \mathbb{R}^{P \times d} $，得到 $ Z^{(i)} \in \mathbb{R}^{P \times d_m} $。

   • 通过线性层将维度从 $ d_m $ 映射到LLM的隐藏维度 $ D $，生成最终的重编程表示 $ O^{(i)} $。

---

**3. 文本原型的作用**
• 语义对齐：文本原型 $ E' $ 是从LLM的词嵌入中提取的少量代表性向量（如“上升”、“下降”、“平稳”等），通过学习这些原型与时间序列模式的关联，将连续信号映射到离散语义空间。

• 效率优化：仅使用 $ V' $ 个原型而非全部词表，减少计算量，同时避免噪声干扰。

---
**注意力机制计算相同模态输入（如文本token与文本token）之间的相关性。通过跨注意力，能计算不同模态之间输入（如文本token与时间序列）之间的相关性，经过多轮的跨注意力，能建立输入时间序列与特定文本token（如增加，减少，趋于平缓，剧烈波动等）之间高维联系，形成可以被传统LLM理解的输入向量。**

---
* 训练的**LLM Body是被冻结的**，也就是不参与反向传播，直接使用预训练的结果。也就是真正被训练的只有时间序列对齐部分。
* TimeLLM解决了时间序列对齐问题，但他仅限于预测数据，只能输入单一变量时间序列，且embadding和归一化过程注定要对数值精度泛化，可能损失数据精度。
#### ChatTS改进
![ChatTS_Structure](/images/ChatTS_Structure.png)
* 由于ChatTS要对多变量时间序列进行分析，必须明确数据序列与文本输入之间的顺序关系。即，**注意输入顺序或文本描述割裂导致的上下文信息丢失​问题。**
  * 如，在分析“HTTP请求”，“CPU利用率”的关联性时，模型需明确两者在输入中的位置关系，文本描述“HTTP请求的周期性”后，直接插入HTTP请求的时间序列编码向量，再跟随文本“对比CPU利用率”，插入对应的编码向量，确保LLM在生成回答时，能通过**位置信息**​​直接关联文本描述与对应时间序列。
* **数字精度保留策略1**：由归一化和词嵌入引起的精度缺失，记录归一化的缩放因子Value Scaling​和偏移量Value Offset，并将其作为文本提示的一部分输入模型。在训练数据中显式加入数值相关任务（如“计算某时间点的原始值”），强制模型学习数值参数与归一化值的关系。通过LLM的数学推理能力，动态还原原始数值。
  * 如，若模型输出“归一化值为0.8，Value Scaling = 100.0, Value Offset = 5.1”，结合参数可还原为 0.8 * 100.0 + 5.1 = 85.1
