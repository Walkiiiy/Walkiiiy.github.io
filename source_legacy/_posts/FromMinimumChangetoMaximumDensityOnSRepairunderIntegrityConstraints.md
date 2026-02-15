---
title: From Minimum Change to Maximum Density:On S-Repair under Integrity Constraints
date: 2025-04-22 19:15:31
tags:
---
**文献阅读:改进的S-repair数据清洗算法：从最小程度改变到保留最大数据密度**
**From"From Minimum Change to Maximum Density: On S-Repair under Integrity Constraints" by Yu Sun，Shaoxu Song**
## 总览
原始的S-repair数据清洗只要求删除尽可能少的数据以恢复完整性约束，实际应用中却可能有多种移除集方案都能满足。本研究提出了保留最大密度的选择方案，该方案是NP完全的，因此又进一步提出了多项式时间内可解的启发式解法作为近似解决方案，该解决方案在特殊情况下可实现多项式时间内返回最优解。
## Pre_Definitions：
* 给定一个关系模式 $ A = (A_1, \ldots, A_m) $，关系实例 $ I = \{t_1, \ldots, t_n\} $ 是模式 $ A $ 上的一组元组。每个元组 $ t_i $ 由属性值组成，即 $ t_i = \{t_i[A_1], t_i[A_2], \ldots, t_i[A_m]\} $，其中 $ t_i[A_j] $ 表示元组 $ t_i $ 在属性 $ A_j $ 上的值。
* DC：Denial Constraint，以否定约束表示实例的完整性约束：
  $$
  \varphi: \forall t_i, t_l \in I, \neg(P_1 \land \cdots \land P_j \land \cdots \land P_J),
  $$
对$I$中每一行元组t，不允许他们同时满足条件$P_1,P_2……$
例如，若有函数依赖规则FD：`Load → Count`
转换为否定约束：  
  $$
  \varphi: \forall t_i, t_l \in I, \neg(t_i[\text{Load}] = t_l[\text{Load}] \land t_i[\text{Count}] \neq t_l[\text{Count}])
  $$  
即禁止两个元组在 `Load` 相同的情况下 `Count` 不同。  
## 最小程度改变的S-Repair
###### S-Repair(optimal subset repair)具体可参见'Computing Optimal Repairs for Functional Dependencies'by ESTER LIVSHITS and BENNY KIMELFELD
![](/images/load-count.png)
如图，当对地铁负载数与载人数建立函数依赖load->count时，会出现违反该函数依赖的脏数据$t_1, t_2, t_3, t_4, t_5, t_6$，若只要求删除的元组数量最少，则有不同的删除修复选择有：
{$t_1, t_2, t_3, t_4, t_5, t_6$},
 {$t_0, t_2, t_3, t_4, t_5, t_6$}, 
 {$t_0, t_1, t_3, t_4, t_5, t_6$}, 
 {$t_0, t_1, t_2, t_4, t_5, t_6$}, 
 {$t_0, t_1, t_2, t_3, t_5, t_6$}.
但是最佳的修复只能为{$t_1, t_2, t_3, t_4, t_5, t_6$}
## 保留最大数据密度的S-Repair
定义数据实例$I$的数据密度：
$$
  \mathcal{D}(I) = \sum_{t_i \in I} \sum_{t_l \in L^k_I(t_i)} \sum_{A_j \in A} (1 - d_j(t_i, t_l)).
  $$  
  - $ 1 - d_j(t_i, t_l) $：距离越小，该项越大，密度越高。  
  - 对所有元组的，K个最近的邻居的，每一个属性的$1-d$求和，得到整体密度。

**最终目的是找到其能保留最大密度D的移除集。**
根据proposition1：
```
The removal set with the maximum densityis always a minimal removal set.
```
**保留最大密度D的移除集一定是最小的移除集**，也就是保留元组数最多的移除集之一。
证明：
  1. 假设存在最大密度移除集 $ I_N^* $，但它不是最小移除集：  
     根据定义，存在另一个移除集 $ I_N' $，满足：  
     - $ |I_N'| < |I_N^*| $，  
     - $ I \setminus I_N' \models \Sigma $。

  2. 构造新的移除集 $ I_N'' = I_N^* \cup I_N' $：  
     - 由于 $ I \setminus I_N^* $ 和 $ I \setminus I_N' $ 均满足约束，它们的交集：  
       $$
       I \setminus (I_N^* \cup I_N') = (I \setminus I_N^*) \cap (I \setminus I_N') 
       $$  
       也满足约束（完整性约束在子集下闭包）。  
     - 因此，$ I_N'' $ 是一个移除集。

  3. 分析移除$ I_N'' $后的密度：  
     - 因为 $ I \setminus I_N'' \subseteq I \setminus I_N^* $，移除 $ I_N'' $ 后剩余的数据更少，可能引入更多低密度元组。  
     - 但根据密度定义，删除更多元组通常不会增加密度
     - 因此，有：  
       $$
       \mathcal{D}(I \setminus I_N'') \geq \mathcal{D}(I \setminus I_N^*).
       $$  
       这与 $ I_N^* $ 是最大密度移除集的假设矛盾。  
     上述矛盾表明，不存在比 $ I_N^* $ 更小的移除集 $ I_N' $。因此，$ I_N^* $ 必须是最小移除集。

因此保留最大密度移除集是最小移除集之一。

由于存在多种可选的最小移除集方案，枚举$I_C$中的每一种方案已经是指数级，且对每一方案都要动态计算移除后的每一元组与其K个近邻之间的距离来计算其密度，因此为NP完全问题，暂无多项式时间解法。
## 基于冲突解决的启发式解法
* **冲突检查**：
 定义$ I_C $ 为包含所有参与约束冲突的元组，即：  
  $$
  I_C = \{ t_i \in I \mid \exists t_l \in I, (t_i, t_l) \not\models \Sigma\}.
  $$
  我们仅处理 $ I_C $ 中的元组，非冲突元组（$ I \setminus I_C $）必然保留
* **密度排序**：
  由于直接计算删除后的密度 $ \mathcal{D}(I \setminus I_N) $ 需动态更新邻居，计算成本高，因此采取**近似方法**：对每个冲突元组 $ t_i \in I_C $，计算其与非冲突元组 $ I \setminus I_C $ 的邻居密度：  
  $$
  \rho_i = \sum_{t_l \in L^k_{I \setminus I_C}(t_i)} \sum_{A_j \in A} (1 - d_j(t_i, t_l)),
  $$
  其中 $ L^k_{I \setminus I_C}(t_i) $ 是 $ t_i $ 在 $ I \setminus I_C $ 中的 $ k $ 最近邻。  
随后将 $ I_C $ 中的元组按 $ \rho_i $ **升序排列**，优先删除密度最低的冲突元组，直到不存在约束冲突。
* **迭代删除**：
最后再检查一遍 移除集$ I_N $ 中每个元组，若移除后不影响约束满足性，则将其移出 $ I_N $。

**不保证结果最佳，但保证多项式可解且结果较优**
## 启发式解法在特殊情况下的最优解：
在特殊环境下，启发式算法可获取最优解。
定义团冲突Clique Conflicts $I_C'$为：
冲突元组集合 $ I_C' \subseteq I_C $ 中的每一对元组均相互冲突，即：
  $$
  \forall t_i, t_l \in I_C', i \neq l \Rightarrow (t_i, t_l) \not\models \Sigma
  $$
则在满足所有冲突元组可以**构成不相交的团冲突集合**，且能选取$k$满足所有元组(包含冲突元组)的**前$k$个近邻都为非冲突元组**，即：
$$
  L_I^k(t_i) = L_{I \setminus I_C}^k(t_i) \quad \forall t_i \in I
  $$
则使用启发式解法可返回最优解。
## Que:
* 论文第一部分提到：
```
For instance, Figure 1 illustrates some
example subway data, which are collected by our partner
company. It collects Load sensor readings in subway trains
and the corresponding passenger Count in factory pressure
testing……Integrity constraints are often employed to identify the errors.
For example, in Figure 1, a functional dependency (FD) Load → Count 
identifies t1, t2, t3, t4 in violation with t0,
which have the same Load 0 but different Count.
```
使用了地铁传感器负载重量与乘客数量的关系实例，当对该实例套用函数依赖FD load->count时，发现有t1, t2, t3, t4违反了该函数依赖
可也许地铁实际运行中load->count本就不该存在函数依赖，同一负载下会存在多种人数可能（不同人的体重差异），该举例也许不够准确...?
* 由于是基于数据密度进行筛选，如果有违反了完整性约束的离群值，但不是脏数据，那几乎一定会被删除掉，感觉某种程度上限制了算法的应用场景。
* 能不能进一步想，若移除后剩余密度最小有好几个移除集方案（都是最优），该如何选择？