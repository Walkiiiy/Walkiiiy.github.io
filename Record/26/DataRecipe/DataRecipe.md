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


综上，我们的主要贡献是：
#### Contributions
1.  提出一种基于能力空间的数据配方一体化框架
2.  提出动态粒度的能力空间，并给出该空间的理论评估标准及构建算法
3.  提出基于能力空间的两种新的数据评估算法
4.  提出在能力空间中综合各种数据评估，在线生成数据配方的策略


## Related Works

#### 目标相关度
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
- R&B

#### 数据

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
#### 4.2 模型能力-数据维度映射
在4.1节的基础上我们构建了与训练目标贴合的模型能力空间，并对模型的目标能力维度参数进行了估计。但要在该空间中构建数据配方，还需要将模型的起始能力参数和数据点映射到该空间中。

#### 4.3 评估反馈-动态配方


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
