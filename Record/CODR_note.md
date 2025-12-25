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



