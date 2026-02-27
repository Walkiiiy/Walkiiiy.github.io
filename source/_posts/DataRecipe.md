---
title: DataRecipe
date: 2026-02-15 21:10:27
tags:
---
# Gradient Receipe: Evolving Selection-Proportion pipline into unified gradient optimization
## Abstract
数据配方指对大模型的预训练或微调数据进行针对性的清洗、选择、配比等操作以最大化提升训练效率。然而，现有的众多数据配方方案都针对特定输入情况和优化目标，不同方案面临输入异构、局部目标无法统一等问题。当存在多个配方目标时，由各自独立的数据操作步骤串联成配方管道，造成整体可用性差、效率低下。其根本原因在于选择配比等配方操作的孤立（motivation）

为了解决这些问题，我们针对选择-配比环节，通过将选择配比管道优化成一个能力空间的建模-映射-策略逼近任务，根据训练过程中的评估反馈在线生成数据配方。

与传统方法相比，我们的方法将离散的数据配方步骤优化成一个整体操作，统筹多种配方指标，并支持从多种不同的输入组合中蒸馏微调目标。

在。。。。的实验中。。。。

## Introduction
数据配方是...
数据选择是...数据配比是...选择-配比...
然而，将零散局部的操作组成管道进行配方，面临以下问题...

针对以上问题，我们提出理想化的解决方案：用一个针对训练目标动态构建的能力空间，精准定位预训练模型和目标微调模型各个维度的能力，并将数据点以多种优化目标尺度降维成在能力空间中的梯度影响，从而将整个数据配方流程抽象成一个梯度优化问题。

然而，该框架面临巨大的理论和技术挑战：

#### Challenge
1. 高质量的能力空间难构建

      - 训练目标是动态的抽象的，难以抽取训练目标
      - 有无数种建模方式，粒度难控制

2. 目标相关度映射难实现

      - 映射精确性，要客观标准反映数据与能力空间的相关度


3. 学习能力映射难实现
      	
     - 映射精确性，要客观标准反映模型对这条数据的学习难度


4. 映射协同性、有效性

      - 协同目标相关度和学习能力映射
      - 不能启发式映射，必须与模型训练效果挂钩
      - 可由训练效果反馈证明


针对以上挑战，我们的主要贡献是：
#### Contributions
1.  提出一种基于能力空间的数据配方一体化框架
2.  提出动态粒度的能力空间，并给出该空间的理论评估标准及构建算法
3.  提出基于能力空间的两种新的数据评估算法
4.  提出在能力空间中综合各种数据评估，在线生成数据配方的策略


## Related Works

#### 能力空间
- CDT: A Comprehensive Capability Framework for Large Language Models Across Cognition, Domain, and Task
    - ACL ARR 2025
    - 能力相关度引导的数据选择
    - 建立了首个基于成熟心理学理论的LLM能力分类学体系
    - 现有任务局限于特定任务或孤立能力，他提出系统化、多维度的模型能力分类与评估框架。（通用性更强）
    - 基于心理学中成熟的 Cattell-Horn-Carroll（CHC）认知理论，将一条指令或一个任务，分解为三个正交的维度：如何做（Cognition）、关于什么（Domain）、做什么（Task）。
    
    - 面向通用能力的数据集质量度量：
        - 覆盖率
        - 多样性
  
    - 能力空间被称为“capability framework”（整个评估框架）和“composite capability”（单个数据点），这个空间是离散的、有限的（约9504个点）
    
    - 只关注能力匹配（数据相关度），不关注能力强弱和数据质量。标注模型只标注不打分
    - 可以借用他的tag部分


- MIG
  - 值得一提的是MIG通过将采样数据动态映射成离散标签，试图对全局语义空间统一建模，

- R&B
- 
#### 目标相关度映射

-  MIG: Automatic Data Selection for Instruction Tuning by Maximizing Information Gain in Semantic Space 
   -  ACL 2025 findings
   -  评级+嵌入相似度级联灵感来源
- Analyzing Similarity Metrics for Data Selection for Language Model Pretraining
    - 单纯嵌入相似度不可取
    - 通用嵌入模型是为了语义理解数据而生，而该映射器重点捕捉的是在数据学习过程中的相关性，两者存在错位，因此需要评级矫正




#### 学习能力映射

- Data Whisperer: Efficient Data Selection for Task-Specific LLM Fine-Tuning via Few-Shot In-Context Learning 
  - ACL main 2025
  - 利用上下文学习能力来做学习能力映射

## 4 Method
### 4.1 构建高质量能力空间
构建数据配方的第一步是要明确训练目标的能力。为了考虑到各种配方目标，我们不仅需要确定能力的维度，还要量化能力之间的相对强弱。
受......（CDT）的启发，我们以构建能力空间的方式抽取这个训练目标。我们将在4.1.1提出能力空间构建的目标并证明其有效性，随后在4.1.2提出满足该目标的动态粒度构建算法

#### 4.1.1：  能力空间概念

    - 我们希望


#### 4.1.2 构建标准及其理论有效性 (Criteria for Construction and Theoretical Validity)

构建能力空间的核心挑战在于如何确保空间划分的有效性，即如何保证在该空间内计算出的数据配方能够逼近理论最优解。受 CDT 框架 中关于数据集质量度量的启发，我们针对领域微调任务，提出了评估能力空间质量的两个核心标准：**覆盖率 (Coverage)** 与 **明确度 (Distinctness)**。

为了证明这两个标准的必要性，我们引入了引理 1 (Lemma 1)，通过量化配方过程中的“后悔值” (Regret)，从理论上证明了能力空间的构建质量直接决定了数据配方的优化上界。

**1. 定义构建标准 (Defining the Criteria)**

*   **覆盖率 (Coverage)**：能力空间必须充分覆盖目标任务所需的技能范围。若将目标模型所需能力集合记为 $\mathcal{G}(T)$，当前空间覆盖的能力集合记为 $\mathcal{G}(M)$，构建目标是最小化两者间的面积差 $|\mathcal{G}(T) - \mathcal{G}(M)|$。覆盖率不足将导致配方在关键维度上的缺失。
*   **明确度 (Distinctness)**：空间内的能力维度划分应具有高区分度。理想的能力空间应使用最少维度的标签精确描述数据，使得同一簇内的数据在梯度方向上高度一致（高密度），而不同簇之间界限分明（高分离度）。

**2. 理论证明：引理 1 (Theoretical Proof: Lemma 1)**

我们通过分析数据配方中的“后悔值”来证明上述标准的有效性。后悔值定义为在两个等大的技能簇 $D_i$ (较优) 和 $D_j$ (较差) 之间，若未能正确交换元素所导致的最大优化收益损失。

**Lemma 1.** 数据配方的后悔值上界由能力簇的**目标密度 (Target Density)** 和 **簇间分离度 (Separation)** 共同决定。当能力空间具备高明确度（即高密度）时，后悔值趋近于零。

*证明：*

首先，对任意能力簇 $D_k$，我们定义以下变量：
*   **平均对齐度 (Average Alignment)** $A_k$：簇内数据梯度与目标任务梯度的平均点积。
*   **簇内最大偏离 (Intra-cluster Deviation)** $r_k$：簇内数据与中心梯度的最大差异，代表簇内的“噪声”或粒度粗糙程度。
*   **目标密度 (Target Density)** $\rho_k = A_k / r_k$：该指标量化了“明确度”。$\rho_k > 1$ 意味着簇内数据的平均有效性显著高于其内部噪声，即该能力维度定义清晰。
*   **簇间分离度 (Separation)** $\delta = A_i - A_j$ (假设 $A_i > A_j$)：衡量两个能力簇在优化贡献上的区分度。

根据 Raju & Brockett (R&B) 的理论，交换两个簇元素的后悔值 $R_S(i, j)$ 存在上界：
$$ R_S(i, j) \leq \max \{0, r_i + r_j - \delta \} $$

将密度公式 $r_k = A_k / \rho_k$ 代入上式，并设 $\rho_{\min} = \min(\rho_i, \rho_j)$，可推导得：
$$ R_S(i, j) \leq \max \left\{ 0, \frac{A_i + A_j}{\rho_{\min}} - \delta \right\} $$

考虑到 $A_i = A_j + \delta$ 且 $A_j \ge 0$，在最坏情况下（上界最紧时），我们有 $A_i + A_j \ge \delta$。代入后提取公因式 $\delta$，得到最终的误差上界公式：

$$ R_S(i, j) \leq \delta \cdot \max \left\{ 0, \frac{1}{\rho_{\min}} - 1 \right\} $$

**3. 结论 (Conclusion)**

上述推导揭示了能力空间构建标准与配方效果之间的数学关系：

1.  **明确度的决定性 (Decisiveness of Distinctness)**：公式表明，只要满足 $\rho_{\min} \ge 1$，即簇内数据高度一致（$A > r$），后悔值上界即归零（$R_S \to 0$）。这证明了“明确度”是构建高质量能力空间的前提——只有当数据被精准映射到高密度的能力簇时，梯度优化才能达到理论最优。
2.  **覆盖率与分离度的协同 (Synergy of Coverage and Separation)**：虽然分离度 $\delta$ 作为系数出现在不等式中，但在高明确度（$\rho \ge 1$）条件下，分离度对误差的影响被消除。然而，为了最大化配方收益，我们需要能力空间尽可能覆盖高价值区域（高 $A$ 值），从而在不同簇之间形成有效的分离。

综上所述，**覆盖率**保证了优化的上限（即存在高价值数据），而**明确度**保证了优化的下限（即能准确识别并利用这些数据）。这为下一节提出以贴合和明确为目标的动态构建算法提供了理论支撑。

#### 4.1.3 ALG1：以贴合和明确为目标的 Capability Space 构建算法 (Algorithm 1: Construction of Capability Space Targeting Coverage and Distinctness)

基于引理 1 (Lemma 1) 的结论，高质量的能力空间必须同时满足**高明确度**（High Distinctness, $\rho_{\min} \ge 1$）和**高覆盖率**（High Coverage）。然而，预定义的静态分类体系（如 CDT 中的固定分类学）难以适配异构的输入数据和动态变化的微调目标。粒度过粗会导致簇内噪声过大（$r_i$ 增加，$\rho_i$ 降低），而粒度过细则导致计算复杂度爆炸且缺乏泛化性。

为此，我们提出一种**动态粒度构建算法 (Dynamic Granularity Construction Algorithm, ALG1)**。该算法不依赖预设标签集，而是通过由底向上的“原子特征提取”与由顶向下的“密度驱动聚类”相结合，自适应地构建能力维度。

###### 1. 算法核心思想 (Core Intuition)

算法的目标是寻找一组最优的技能簇划分 $\mathcal{C} = \{D_1, D_2, \dots, D_m\}$，使得空间整体的后悔值上界最小化。根据 Lemma 1，这等价于在保证覆盖率的前提下，最大化所有簇的目标密度 $\rho$。

我们将构建过程形式化为从细粒度技能描述集 $S = \{x_1, x_2, \dots\}$ 到粗粒度技能簇 $\{D_k\}$ 的映射优化问题。算法包含两个阶段：**原子化画像生成 (Atomic Profiling)** 与 **动态密度聚类 (Dynamic Density Clustering)**。

###### 2. 算法步骤 (Algorithm Steps)

**阶段一：原子化画像生成 (Phase I: Atomic Profiling)**
首先解决输入异构性问题。利用大语言模型（LLM）强大的 Pattern Recognition 能力，对训练集中的每个数据点 $d_i$ 进行零样本（Zero-shot）标签提取，生成细粒度的原子技能描述 $x_i$。
*   **输入**：原始数据集 $\mathcal{D}$，目标模型描述 $T$。
*   **操作**：$\text{Tag}(d_i) \to x_i$（例如："Physics", "Calculus", "Logic"）。
*   **输出**：原子技能空间 $S$。

**阶段二：动态密度聚类 (Phase II: Dynamic Density Clustering)**
在原子空间 $S$ 上，通过合并相似标签形成初始簇，并基于 Lemma 1 定义的密度指标进行迭代优化。

*   **Step 1: 初始化 (Initialization)**
    将 $S$ 中的标签嵌入向量空间，执行层次聚类（Hierarchical Clustering）得到初始簇集合 $\mathcal{C}^{(0)}$。

*   **Step 2: 密度检测与分裂 (Density Check & Split - Ensuring Distinctness)**
    对于每个簇 $D_k \in \mathcal{C}$，计算其目标密度 $\rho_k = A_k / r_k$。
    *   若 $\rho_k < 1$（即 $r_k > A_k$），说明该簇内部方差过大，包含的技能过于杂乱，无法明确指导梯度方向。
    *   **操作**：将 $D_k$ 分裂为子簇 $D_{k1}, D_{k2}$（细化粒度），直到所有子簇满足 $\rho \ge 1$ 或达到最小粒度阈值。

*   **Step 3: 分离度检测与合并 (Separation Check & Merge - Reducing Redundancy)**
    计算任意两个簇之间的分离度 $\delta_{ij} = |A_i - A_j|$。
    *   若 $\delta_{ij} < \epsilon$（$\epsilon$ 为小量），说明两个簇在目标任务上的梯度贡献高度相似，区分它们的边际收益极低。
    *   **操作**：合并 $D_i$ 和 $D_j$（粗化粒度），以减少维度冗余并提升计算效率。

*   **Step 4: 覆盖率对齐 (Coverage Alignment)**
    计算当前空间 $\mathcal{G}(M)$ 与目标需求 $\mathcal{G}(T)$ 的面积差。通过引入目标描述 $T$ 的关键词向量，剔除与 $T$ 正交（即 $A_k \approx 0$）的无关簇，确保空间专注于有效区域。

###### 3. 算法伪代码 (Pseudo-code)

```latex
\begin{algorithm}
\caption{Dynamic Capability Space Construction (ALG1)}
\begin{algorithmic}
\Require Dataset $\mathcal{D}$, Target Description $T$, Density Threshold $\tau \approx 1$
\Ensure Optimized Capability Clusters $\mathcal{C}_{opt}$

\State \textbf{Phase I: Atomic Profiling}
\State $S \leftarrow \emptyset$
\For{$d_i \in \mathcal{D}$}
    \State $x_i \leftarrow \text{LLM\_Extract\_Tags}(d_i, T)$ \Comment{Extract fine-grained skills}
    \State $S \leftarrow S \cup \{x_i\}$
\EndFor

\State \textbf{Phase II: Dynamic Clustering}
\State $\mathcal{C} \leftarrow \text{Initial\_Clustering}(S)$
\Repeat
    \State $Stable \leftarrow True$
    \For{$D_k \in \mathcal{C}$}
        \State Calculate alignment $A_k$ and deviation $r_k$
        \State Calculate density $\rho_k = A_k / r_k$
        \If{$\rho_k < \tau$} \Comment{Violates Lemma 1 distinctness}
            \State $\mathcal{C} \leftarrow (\mathcal{C} \setminus \{D_k\}) \cup \text{Split}(D_k)$
            \State $Stable \leftarrow False$
        \EndIf
    \EndFor
    \For{$D_i, D_j \in \mathcal{C}$}
        \If{$|A_i - A_j| \approx 0$} \Comment{Low separation}
            \State $\mathcal{C} \leftarrow \text{Merge}(D_i, D_j)$
            \State $Stable \leftarrow False$
        \EndIf
    \EndFor
\Until{$Stable$ is True}
\State \Return $\mathcal{C}$
\end{algorithmic}
\end{algorithm}
```

###### 4. 总结 (Summary)
ALG1 通过将 Lemma 1 的理论约束转化为“分裂-合并”的动态过程，解决了传统方法中粒度难控制的问题。该算法确保了生成的每个能力维度在梯度优化意义上都是“明确的”（Distinct），从而为后续的目标相关度映射（Mapping）和数据配方计算奠定了坚实的结构基础。


### 4.2 模型能力-数据维度映射框架

在 4.1 节的基础上，我们构建了与训练目标贴合的模型能力空间 $\mathcal{C}_{opt}$。但要在该空间中生成数据配方，必须将异构的原始数据点映射为该空间内的连续特征向量。

现有基于嵌入模型（Embedding）或大型语言模型（LLM）的评估策略往往面临计算成本与评估深度的矛盾。为了统筹不同的数据评估策略并保证微调过程的高效性，我们首先规定统一的映射器标准，并在此基础上提出一种基于级联架构的语义相关度映射算法。

#### 4.2.1 映射器标准 (Standard for Capability Mappers)

为了确保数据配方框架在海量数据集上的可扩展性（Scalability）与可插拔性（Plug-and-play），任何合法的数据映射算法必须严格遵循以下三项接口与性能约束：

**1. 标准化输入 (Standardized Input)**
映射过程被形式化为一个映射函数 $\mathcal{M}_{\phi}$ 对元组的求解：$(d, \mathcal{C}_{opt}) \xrightarrow{\mathcal{M}_{\phi}} \mathbf{v}_d$。

* **数据点 $d$**：来自原始微调或预训练语料库 $\mathcal{D}$ 的单条数据。
* **能力空间 $\mathcal{C}_{opt}$**：由上一节动态构建的高明确度空间，包含 $m$ 个正交的能力维度簇 $\{C_1, C_2, \dots, C_m\}$。
* **辅助模型参数 $\phi$**：映射算法所依赖的外部评估模型（如文本向量化模型或预训练 LLM）。

**2. 标准化输出与不确定性量化 (Standardized Output and Uncertainty)**

* 映射器必须输出一个定长的 $m$ 维实数特征向量 $\mathbf{v}_d = [v_{d,1}, v_{d,2}, \dots, v_{d,m}]^\top \in \mathbb{R}^m$。
* **值域与物理意义**：各分量 $v_{d,j}$ 需经过 Min-Max 归一化至 $[0, 1]$ 区间。若存在评估不确定性或多义性，该数值代表模型对数据 $d$ 属于能力 $C_j$ 的置信度期望（Expected Confidence）。

**3. 时间复杂度约束 (Time Complexity Constraint)**

* **推论逻辑**：若映射算法对数据的评估需要逐一遍历 $m$ 个能力维度，其单条数据的处理时间将达到 $\mathcal{O}(m \cdot t_{aux})$（其中 $t_{aux}$ 为辅助模型 $\phi$ 进行单次前向推理的时间）。在动态粒度空间（$m$ 庞大）与千万级语料库交织的场景下，计算开销将远超大模型训练本身的成本，导致配方失去工程可行性。
* **约束条件**：因此，我们规定：**一个合格的映射算法，其处理单条数据的最大时间复杂度必须严格限制在 $\mathcal{O}(c \cdot t_{aux})$**（$c$ 为极小常数），即时间复杂度必须**独立于能力空间的维度数 $m$**。这就要求映射器必须具备隐式并行推理或一次性路由（Single-pass Routing）的能力。

---

#### 4.2.2 基于级联架构的语义相关度映射算法 (Cascade Semantic Relevance Mapping Algorithm)

单凭现成的嵌入模型（Off-the-shelf Embeddings）在数据选择中往往表现不佳。根据近期 NeurIPS 2025 的研究论证，传统文本相似度度量（Similarity Metrics）难以准确捕捉数据对语言模型训练动态（Training Dynamics）和深层逻辑泛化的真实影响（参考来源：*Analyzing Similarity Metrics for Data Selection for Language Model Pretraining*, NeurIPS 2025）。然而，纯粹依赖 LLM 进行全维度的深层逻辑校验又会打破 4.2.1 节设定的 $\mathcal{O}(c \cdot t_{aux})$ 复杂度上限。

针对这一矛盾，我们提出一种由粗到细的级联映射架构（Coarse-to-Fine Cascade Architecture），结合嵌入空间的低复杂度路由与 LLM 的深度逻辑校验。

**Step 1: 稠密检索预筛 (Dense Routing via Embedding Space)**

* 利用轻量级 Embedding 模型将数据 $d$ 投影至连续特征空间，计算其与所有 $m$ 个簇心 $\{C_1, \dots, C_m\}$ 的余弦相似度。
* 选取 Top-$K$ 个最高相似度的簇（如 $K \le 3$）作为候选能力维度，其余维度直接置零。由于仅涉及向量内积检索，此步骤满足极低的时间复杂度。我们将这 $K$ 个簇的归一化相似度记为空间标尺 $S_{emb}(d, C_k) \in (0, 1]$。

**Step 2: 预训练模型深度校验 (Deep Verification via LLM)**

* 将数据 $d$ 与 Top-$K$ 个候选能力描述构建为提示词，调用 LLM 进行单次零样本（Zero-shot）打分。该操作时间复杂度被严格控制在 $\mathcal{O}(1 \cdot t_{aux})$。
* LLM 输出一个离散的逻辑门控置信度 $W_{llm} \in \{0, w, 1\}$，分别对应无关、弱相关与强相关。其中 $w \in (0, 1)$ 为弱相关阻尼参数（本文实验中默认取 $w = 0.1$），用于保留连续空间中的微小梯度贡献，避免梯度过度稀疏化。

**Step 3: 乘积融合映射 (Multiplicative Fusion Mapping)**
我们将最终的能力映射值 $v_{d,k}$ 定义为空间标尺与逻辑门控的乘积：


$$v_{d,k} = S_{emb}(d, C_k) \cdot W_{llm}(d, C_k)$$

* **推论逻辑与理论依据**：该融合公式本质上是一种带有软门控（Soft-gating）的迟融合机制。根据最新的语义空间信息增益理论（参考来源：*MIG: Automatic Data Selection for Instruction Tuning by Maximizing Information Gain in Semantic Space*, ACL Findings 2025），数据点对其关联能力维度的信息贡献量，应按比例（in proportion to）由其数据质量置信度决定。
* 在此公式中，LLM 的逻辑校验 $W_{llm}$ 作为过滤“词汇共现陷阱（Lexical Co-occurrence Trap）”的硬性掩码（Mask）；而 $S_{emb}$ 则保留了同等逻辑置信度下，数据在特征空间中的连续距离差异。为了进一步克服嵌入向量固有的各向异性（Anisotropy）导致的数值拥挤问题，引入温度超参数 $\tau$ 对相似度进行对比 Softmax 放缩：

$$v_{d,k} = \left( \frac{\exp(S_{emb}(d, C_k) / \tau)}{\sum_{j \in TopK} \exp(S_{emb}(d, C_j) / \tau)} \right) \cdot W_{llm}(d, C_k)$$


#### 4.2.3 基于维度条件上下文困惑度的学习能力映射算法

模型能力-数据维度映射的另一核心挑战在于量化数据本身的“固有学习价值”。为了客观反映数据在深层语义逻辑上的“可习得性（Learnability）”并契合教育学中的“最近发展区”理论，本节提出一种基于**维度条件上下文困惑度偏差 (Dimension-Conditioned Contextual Perplexity Deviation, DC-CPD)** 的学习能力映射算法。

为严格满足 4.2.1 节设定的 $\mathcal{O}(c \cdot t_{aux})$ 时间复杂度约束并生成多维特征向量，本算法复用了 4.2.2 节的稠密路由结果，避免了全局计算与高昂的检索开销。

##### 1. 代理参考模型与基准对齐

直接使用目标微调模型进行数据评估会带来极高的计算负担。受 Data Whisperer 弱到强（Weak-to-strong）评估范式的启发，我们引入一个能力稳定且与目标模型同源的轻量级基座模型作为**参考模型 $\Theta_{ref}$**（例如使用 3B 模型代理 7B 模型）。这不仅将推理耗时压缩至最小，同时保证了评估分布与目标微调模型的高度对齐。

##### 2. 全局随机基线（绝对认知难度期望）

为了剥离 In-Context Learning (ICL) 中因格式提示或上下文长度带来的非能力增益，我们摒弃了传统的 Zero-shot 基线，转而构建**全局随机上下文期望**。
对于数据 $d$，我们从全局数据集 $\mathcal{D}$ 中随机采样 $N$ 组（如 $N=3$），每组包含 $k$ 个样本的随机上下文 $E_{rand}^{(i)}$。定义数据 $d$ 的全局基准预测损失 $\mathcal{L}_{base}(d)$ 为：


$$\mathcal{L}_{base}(d) = \mathbb{E}_{E_{rand} \sim \mathcal{D}} [ \mathcal{L}(d | \Theta_{ref}, E_{rand}) ] \approx \frac{1}{N} \sum_{i=1}^N \mathcal{L}(d | \Theta_{ref}, E_{rand}^{(i)})$$


该指标表征了在缺乏特定领域知识引导下，参考模型对数据 $d$ 的绝对认知难度。

##### 3. 维度条件下的认知下界与偏差量化

利用 4.2.2 节语义映射输出的 Top-$K$ 激活候选簇集合 $S_{active}(d)$，我们仅对高度相关的少数维度进行深度学习能力评估。
对于任意候选能力簇 $C_j \in S_{active}(d)$：

* **免检索上下文构造**：直接从簇 $C_j$ 中随机抽取 $k$ 个代表性高质量样本构成条件上下文 $E_j$。此操作的时间复杂度为 $\mathcal{O}(1)$。
* **条件认知下界**：计算参考模型在特定能力 $C_j$ 引导下的预测损失 $\mathcal{L}_{few}^{(j)}(d) = \mathcal{L}(d | \Theta_{ref}, E_j)$。

据此，我们定义数据 $d$ 在能力维度 $j$ 上的**静态上下文困惑度偏差** $\delta_{CPD}^{(j)}(d)$ 为：

$$\delta_{CPD}^{(j)}(d) = 
\begin{cases} 
\mathcal{L}_{base}(d) - \mathcal{L}_{few}^{(j)}(d), & \text{if } C_j \in S_{active}(d) \\
0, & \text{if } C_j \notin S_{active}(d) 
\end{cases}$$

**推论逻辑**：若数据 $d$ 具有显著的上下文增益（$\delta_{CPD}^{(j)} \gg 0$），证明其内部逻辑自洽且能通过 $C_j$ 维度的知识被有效推导，属于位于该维度“最近发展区”的高价值样本。反之，若在强上下文引导下损失依然极高且无明显下降（$\mathcal{L}_{few}^{(j)} \approx \mathcal{L}_{base}$），则大概率为该维度下的不可习得噪声或错误标注。

##### 4. 软截断算子与映射向量生成

为了将上述偏差转化为 $[0, 1]$ 之间且对梯度优化友好的标准化评分，我们设计了包含软惩罚机制的非线性映射算子。对每一个维度 $j$，其学习能力分数 $r_{learn, j}(d)$ 计算如下：

$$r_{learn, j}(d) = \text{Sigmoid}\left(\frac{\delta_{CPD}^{(j)}(d) - \mu_j}{\sigma_j}\right) \cdot \left[ 1 - \text{Sigmoid}\left(\frac{\mathcal{L}_{few}^{(j)}(d) - \tau_{noise}}{\gamma}\right) \right]$$

其中，$\mu_j, \sigma_j$ 为能力簇 $C_j$ 内历史偏差的统计均值与方差；右侧惩罚项为噪声过滤函数，$\tau_{noise}$ 为经验损失阈值，$\gamma$ 控制衰减平滑度。该算子不仅奖励高上下文增益的数据，还通过平滑衰减机制，在不引起梯度突变的前提下，有效压制了那些在 Few-shot 下依然高损耗的顽固噪声。

最终，映射器输出一个 $m$ 维稀疏特征向量 $\mathbf{r}_{learn} \in \mathbb{R}^m$。结合 4.2.2 节的语义相关度向量 $\mathbf{v}_d$，我们即可在 4.3 节中通过张量运算，精确刻画单条数据对目标模型的综合梯度影响。


### 4.3 评估反馈-动态配方
本节着重解决两个核心问题：
1. 如何联合以上多种数据评估策略
2. 如何反馈下游模型的实际训练效果到上游的评估策略

....(权重矩阵构建)

模型参数随训练过程动态变化，模型所需的数据类型也会随之变化，意味着权重矩阵需要随整个训练过程动态更新。并且由于该矩阵的信息量较大，这个动态更新必须由足够细粒度的训练反馈信号驱动。传统以batch为单位直接使用验证集进行反馈在这里不可行。

为了解决以上问题，能从单条数据的粒度指导上游数据各评估策略的权重权重更新：
最初的能力空间构建使用的就是一个小验证集
这个验证集实际参与构建的部分（参与层次聚类的干净数据）都属于上层的至少一个簇（也就是能力维度）
对m个能力维度，每个都取其前k个最相关的数据，组成能力验证集（可以复用前面相关度映射结果）
每一个batch先对能力验证集进行训练，不反向传播，只获取每个能力维度的能力验证集在本batch的平均梯度，为第t轮的m个能力梯度。该梯度模长用于指导能力维度权重贝塔更新
然后正式开始此batch的训练，训练时收集每条数据的梯度。本batch训练完用该梯度和每个能力维度的能力梯度的向量相似程度的top k（k由关联的能力维度数量动态决定）来计算该条数据的价值，作为反馈信号指导前面的评估策略权重阿尔法更新
算法：
```
```
## Experiment




<!-- 
## Legacy
- **Intro**
- 从四个角度切入问题
  - proportion only方法是第一个切入角度：
    - How can they be sure if the default domain clustering is best? 怎么就敢确定默认领域划分就是最好的？
    - 没有好的划分就没有好的配比,划分-配比应该是联合优化问题
  - Shallow Clustered by Skills是第二个切入角度：
    - 浅聚类在默认语义向量下，默认会以垂直技能角度划分数据，凭什么假定从默认空间表示的角度（技能）划分是最好的？
	  - 应该在每一轮训练时，模型需要从哪个角度划才怎么划(用深度聚类捕捉聚类的不同角度)
  - One more step: Why one angle at a time? 是第三个切入角度
    - 为什么要固定同一批数据只能有唯一的最优聚类角度？
    - 应该不同角度，多层次聚类
  - 性能和划分-配比粒度是第四个切入角度

- 本研究认为：
  - 聚类和配比应该是动态绑定的，在训练过程中一起更新动态迭代，对数据选择过程来说同属于一个原子操作
- 由此提出自顶向下的聚类配比树作为新的解决方案（拟），希望同时解决：
    - 1. 数据的划分-配比联合优化
    - 2. 找到每一批次数据最优划分角度下的深度聚类（新）
    - 3. 同一批次数据能有不同角度的多层聚类（新）
    - 4. 性能问题



- **Related works**:
    - Proportion:
        - Proportion only:
            - DoReMi: Optimizing Data Mixtures Speeds Up Language Model Pretraining
                - NeurIPS 2023
                - Small model simulating a LLM's preferanced training proportion
                - No ganrante that small model has same preferance as a big model
            - DATA MIXING LAWS: OPTIMIZING DATA MIXTURES BY  PREDICTING LANGUAGE MODELING PERFORMANCE
                - ICLR 2025
        - Fixed semantic clustering & Proportion:
            -  CLIMB: CLustering-based Iterative Data Mixture Bootstrapping for Language Model Pre-training
                - Nvidia https://research.nvidia.com/labs/lpr/climb/
            -  R&B: Breaking the Data Mixing Bottleneck with Just 0.01% Overhead
                - ICML dig-bug long 2025
             
- **Problem formation**:



    **Given:**
    1.  A dataset, \( D \), which is un-categorized or less-categorized.
    2.  A downstream fine-tuning task \( \mathcal{T} \) with a loss function \( \mathcal{L}_{\mathcal{T}} \).

    **Define:**
    - Let \( \mathcal{S} \) be a **clustering strategy**. This strategy includes both:
        -  The number of clusters \( k \).
        -  A set of deep clustering model parameters \(ϕ\).
    
        The strategy takes the dataset \( D \) and ouputs The resulting domains (clustering) of the data, \( D_{\theta} = \{C_1, C_2, ..., C_k\} \).
    - Let \( \mathcal{A} \) be the space of all possible such clustering strategies.

    **Objective:**
    $$
     \min_{m,s} \min_{p^1...p^m-1} \mathcal{L_{eval}} 
    $$
    **Mathematical Formulation:**


 -->

