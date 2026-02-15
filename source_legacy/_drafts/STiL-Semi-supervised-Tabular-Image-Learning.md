---
title: STiL:Semi supervised Tabular-Image Learning
date: 2025-05-28 13:15:01
tags:
---
**文献阅读：大语言模型推荐表格分析策略并生成代码和结果**
**from"TablePilot: STiL: Semi-supervised Tabular-Image Learning for Comprehensive Task-Relevant Information Exploration in Multimodal Claasification"by Siyi Du, Xinzhe Luo, Declan P. O’Regan, Chen Qin**
## 总览
* 自监督学习SSL：可用于无标注数据，但预训练过程与 
* 半监督学习SemiSL：结合标记和未标记数据
* 本文提出STiL：
## 背景
* 目前的大模型可以通过解读扫描图像+表格模态数据，来进行疾病诊断等。但是实际训练中由于缺少标注的数据集（尤其是对罕见疾病分类时）。
* 为了解决标注数据集稀缺问题，传统自监督方案首先利用自监督学习SSL在大规模未标记数据集上预训练模型，随后使用标记数据对下游任务进行有监督微调
![STiL_traditionSSl](/images/STiL_traditionSSl.png)
* 但是仍旧存在问题。其一，基于未标记数据的预训练过程与任务无关，限制了模型捕捉下游任务特定信息的能力（即预训练使用的数据和分类任务本身无关）；其二，微调阶段仅依赖有限标记数据，增加了过拟合风险并削弱了模型的泛化能力。
* 半监督学习SemiSL可以联合少量标记数据和大量未标记数据提取任务相关信息。
* 模态信息鸿沟Modality Information Gap：早期半监督学习方法仅关注模态共享特征（如图像与表格中的心室容积），忽略模态特定特征（如图像中的形状特征、表格中的脉搏率）
![STiL_modality_shared_characteristics](/images/STiL_modality_shared_characteristics.png)
## 半监督学习
https://www.bilibili.com/video/BV1hg4y1A7VQ/?spm_id_from=333.337.search-card.all.click&vd_source=2efc47dd2579af2638d4db7bffb9d2a9
有一堆未标记数据和很少已标记数据时，可以使用半监督学习
**假设：**
* 连续性：相近的数据点大概率有相同的标签 
* 集群：同一标签的数据会集群，决策边界在集群之间
* 流型假设：高维数据可降维
**方法：**
* 一致性正则化：对标注数据点$x_1$半径$r$内的无标注数据点$x_2$加相同的标注，根据模型输出数据$y_2$与$y_1$之间的距离和$y_1$本身的loss计算$x_2$的loss。
* 生成对抗网络
* 主动学习
## STiL架构
![STiL_arch](/images/STiL_arch.png)