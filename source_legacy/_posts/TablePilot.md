---
title: TablePilot
date: 2025-05-25 14:17:14
tags:
---
**文献阅读：大语言模型推荐表格分析策略并生成代码和结果**
**from"TablePilot: Recommending Human-Preferred Tabular Data Analysiswith Large Language Models"by Deyin Yi, Yihao Liu, Lang Cao,Mengyu Zhou, Haoyu Dong, Shi Han, Dongmei Zhang**
## 总览
* 输入表格，大模型自动分析表格背景信息、依赖关系等，推荐**具有分析价值**的数据分析策略，并给出代码和结果（三元组 $<query, code, result>$）。
![无需上下文、用户画像，输入表格，模型推荐好的分析问题并生成代码自问自答](/images/tablePilot.png)
## 传统方案与挑战
#### 传统方案：
* **表格基础数据分析**
用户给定查询，模型提供答案，或者提取子表。这些方法仅根据给定查询返回结果，无法自动生成自然语言查询。
* **表格数据可视化**
* **表格数据建模预测**

尽管这些方案都有比较好的方案，但**目前没有统一的系统能集成表格分析、可视化和统计建模**，自主分析策略并给出结果。
#### 主要挑战：
* 表格数据通常规模大且数据密集，LLMs 难以有效处理。长上下文窗口可能引发幻觉（包括事实型和忠实型幻觉），导致结果不准确。
  - (Lei Huang, Weijiang Yu, Weitao Ma, Weihong Zhong,Zhangyin Feng, Haotian Wang, Qianglong Chen,Weihua Peng, Xiaocheng Feng, Bing Qin, and Ting Liu. 2024. **A survey on hallucination in large language models: Principles, taxonomy, challenges, and open questions**. ACM Transactions on Information Systems.)
  - 解决方案：Analysis Preparation阶段的采样技术
* 现有方法主要执行预定义的数据分析查询以获取结果，这些方法缺乏多样性，无法提供全面分析。
  - 解决方案：模块化分析Module-based Analysis，整合三个
* 分析结果要符合人类认知模式。系统应推荐多样化的数据分析操作，匹配用户的分析偏好。
  - 解决方案：Rec-Align 方法，通过直接融入人类偏好进一步提升分析质量。
## Procedure
TablePilot的流程设计比较繁杂，原文又讲得很省略。好在附录给了一份8页的流程示例。
![procedure](/images/TablePilotProcedure.png)
假设有初始表格T：
![TablePilotOriginTable](/images/TablePilotOriginTable.png)
#### I.Analysis Preparation
* 通过对表格进行**子集采样**并**生成表格解释**来为模型聚焦表格数据。
* 子集采样就是从中选取代表性的数据：
$$ \text{Sampling}(\mathbb{T}_{a \times b}) = \mathbb{T}'_{a' \times b'}$$  
* 生成解释，如表格主题、列描述以及不同列之间的关系，这些解释均用于指导后续分析：
$$ \text{Explanation}(\mathbb{T}) = E.$$
如表格T提取sample为：
```
- 'Airport Code' (Column A): ['EWR', 'JFK', 'LGA', 'LGA', 'SWF', ... ]
- 'Year' (Column B, Numeric): [2011, 1995, 1981, 1999, 1978, ... ]
- 'Month' (Column C): ['Dec', 'Sep', 'Nov', 'Dec', 'Mar', ... ]
- 'Domestic Passengers' (Column D, Numeric): [1901523, 1904407, 1085316, 1096845, 1594280, ... ]
- 'International Passengers' (Column E, Numeric): [2252910, 1102018, 104142, 136952, 14095, ... ]
- 'Total Passengers' (Column F, Numeric): [775695, 2499703, 1858392, 2318716, 1756044, ... ]
```
总结的explaination：
```
{ "Table Theme": "Annual and Monthly Passenger Traffic Statistics for Major U.S. Airports",
"Columns": {
"Airport Code": "This column contains the unique three-letter codes representing different airports. It is essential
for identifying the specific airport associated with the passenger data.",
"Year": "This column indicates the year in which the passenger data was recorded. It helps in analyzing trends and
changes in passenger traffic over different years.", 
......}
"Column Relationships": {
"Airport Code": "The 'Airport Code' column is linked with all other columns to provide context for the passenger
data, allowing for airport-specific analysis of trends and patterns.",
......} }
```
#### II.Module-based Analysis 
* 对采样后的表格 $ T' $ 及其对应的表格解释 $ E $ 进行基于模块的分析（基础分析，数据可视化，统计建模三个模块）。每个模块将 $ T' $ 和 $ E $ 作为输入，生成有意义的查询-代码对 $(q, c)$:
$$ \mathcal{M}_k(\mathbb{T}', E) = (q_k, c_k) $$  
其中 $ k \in \{ba, dv, sm\} $ 代表基础分析（ba）、数据可视化（dv）和统计建模（sm）三种不同的分析任务。
如表格T生成的三种类型的提问以及代码：
1.Basic Analysis Module:
```
[ { "query": "Filter the data to show only the records for the airport code 'JFK’.”,
 "code": "import pandas as pd\nfile = 'table_name.csv'\ndf = pd.read_csv(file)\njfk_data = df[df['Airport Code'] ==
'JFK']\ncsv_file_path = 'output.csv'\njfk_data.to_csv(csv_file_path, index=False)" },
{ "query": "Sort the data by 'Total Passengers' in descending order.",
 "code": "import pandas as pd\nfile = 'table_name.csv'\ndf = pd.read_csv(file)\nsorted_data =
df.sort_values(by='Total Passengers', ascending=False)\ncsv_file_path = 'output.csv'\nsorted_data.to_csv(csv_file_path,
index=False)" },......]
```
2.Table Visualization Module
```
[ { "query": "Generate a line chart to show the trend of total passengers over the years for each airport.",
 "code": "import matplotlib.pyplot as plt\nimport pandas as pd\nfile = 'table_name.txt'\ndf = pd.read_csv(file,
sep='\\t')\nfor airport in df['Airport Code'].unique():\n airport_data = df[df['Airport Code'] == airport]\n
plt.plot(airport_data['Year'], airport_data['Total Passengers'], label=airport)\nplt.xlabel('Year')\nplt.ylabel('Total
Passengers')\nplt.title('Trend of Total Passengers Over the Years for Each
Airport')\nplt.legend()\nplt.xticks(rotation=45)\nplt.tight_layout()\n#Chart INFO: {'x_fields': 'Year', 'y_fields': ['Total
Passengers'], 'chart_type': 'lineChart'}\nplt.show()" }, ......]
```
3. Stastics Modeling Module:
```
[ { "query": "Perform a trend prediction analysis to forecast the total number of passengers for the next 12 months
at JFK airport using historical data.",
 "code": "import pandas as pd\nfrom statsmodels.tsa.statespace.sarimax import SARIMAX\n\nfile =
'table_name.csv'\ndf = pd.read_csv(file)\n\ndf['Date'] = pd.to_datetime(df['Year'].astype(str) + '-' + df['Month'] + '-
01')\njfk_data = df[df['Airport Code'] == 'JFK'].sort_values('Date')\njfk_data.set_index('Date', inplace=True)\n\nmodel =
SARIMAX(jfk_data['Total Passengers'], order=(1, 1, 1), seasonal_order=(1, 1, 1, 12))\nmodel_fit =
model.fit(disp=False)\n\nforecast = model_fit.forecast(steps=12)\nforecast_df = pd.DataFrame({'Date':
pd.date_range(start=jfk_data.index[-1] + pd.DateOffset(months=1), periods=12, freq='M'), 'Forecasted Total Passengers':
forecast})\n\nprint(forecast_df)" }, ......]
```
#### III.Analysis Optimization
首先执行代码以获取每个代码 $ c_k $ 的结果 $ r $：  
$$ \text{Execution}(\mathbb{T}, c_k) = r = 
\begin{cases} 
T, & \text{if } k = \text{ba} \\
V, & \text{if } k = \text{dv} \\
M, & \text{if } k = \text{sm} 
\end{cases} \quad (6) $$  
$ T $ 表示基础分析中数据操作后的子表，$ V $ 表示数据可视化结果，$ M $ 对应统计建模的输出。数据可视化结果 $ r = V $ 也是图像 $ r = I $，后续将作为LLMs视觉模块的输入。
如表格T的三种module code执行结果：
![TablePilotCodeRes](/images/TablePilotCodeRes.png)
其中数据可视化模块代码的执行结果中，难免会出现label重叠等低质量不清晰图像：
![TablePillotunclearChart](/images/TablePillotunclearChart.png)
随后，将查询 $ q $、代码 $ c $ 和结果 $ r $ 组合为分析三元组 $ a = (q, c, r) $。若code执行结果为 $r = \text{Error}$，即表示代码执行出错。

接下来，基于表格采样 $ \mathbb{T} $ 和解释 $ E $ 的结果对分析三元组 $ a $ 进行优化。
方案A：可执行code利用LLMs提升查询和代码与数据及分析意图的对齐度，确保结果更准确且有意义。
方案B：若有包含Error的不可执行code，分析错误原因并加强。生成的图像送入多模态模型处理，看是否有unclear，overlap情况。
优化后，执行优化后的代码生成最终增强的执行结果，得到优化后的三元组 $ a' = (q', c', r') $：  
$$ a' = 
\begin{cases} 
\text{Optimize}_A(q, c, r \mid \mathbb{T}, E), & \text{if } r \neq \text{Error} \\
\text{Optimize}_B(q, c, r \mid \mathbb{T}, E), & \text{if } r = \text{Error} 
\end{cases} \quad$$
比如案例T中的unclear图像optimize方案及结果对比图：
![TablePilotOptimize](/images/TablePilotOptimize.png)
#### IV.Analysis Ranking
根据相关性,多样性,六个维度对每个三元组的query进行评分。这些评分随后被聚合以计算综合得分$s$。根据这些得分，三元组按降序排列，从而选出前 k 个结果：
$$A'_k = \text{Top}_k\left(\text{Rank}\left(\{(q'_i, c'_i, r'_i)\}_{i=1}^{n}\right)\right) \quad $$
如示例T的高分召回query，每一个维度都要LLM分别打分并说明理由：
![TablePioltRanking](/images/TablePioltRanking.png)
## 训练方法
训练策略包含以下三个关键部分：  
- **分析SFT**：训练三个分析模块（$M_{ba}, M_{dv}, M_{sm}$）中的LLM，提升其遵循指令的能力，生成相关查询和准确代码，从而提高分析的相关性，准确性，以及代码的运行成功率。  
- **排序监督微调SFT**：训练排序模块中的LLM，使其更好地根据综合标准评估每个分析三元组并分配适当分数。
- **排序直接偏好优化DPO**：通过DPO实现Rec-Align，优化排序模块中分析三元组的评估，确保评估和评分更贴近人类分析偏好。
  - 先收集人类标注的偏好数据，即对于同一表格的多个分析结果，标注哪些更符合人类偏好（排序）
  - 构造DPOloss函数：
$$
\mathcal{L}(\theta) = -\mathbb{E}_{\left(x, y^{+}, y^{-}\right)}\left[\log \sigma\left(\beta \cdot \log \frac{\pi_{\theta}\left(y^{+} | x\right)}{\pi_{\theta}\left(y^{-} | x\right)}\right)\right]
$$    
$\theta$：模型内部参数。  
$\pi_{\theta}(y|x)$：模型在输入 $x$ 下生成输出 $y$ 的概率。  
$(x, y^+, y^-)$：偏好训练样本，其中 $y^+$ 是人类偏好的输出，$y^-$ 是不偏好的输出。  
$\sigma(\cdot)$：sigmoid函数，将对数概率比值转换为概率值。  
$\beta$：缩放参数，控制偏好分离的程度（$\beta>0$ 时，增大 $\beta$ 会强化对偏好样本的区分）。  
$\mathbb{E}$为期望
  - **通过将loss函数集成到排序模块，训练模型排序时更符合人类偏好，即Rec-Align。**

## 评估指标与结果
* 执行率： 即给定code能正确执行的概率。
![TablePilotEval1](/images/TablePilotEval1.png)
* 召回率：Recall@All 检查正确结果是否存在于整个候选集中，而 Recall@5 和 Recall@3 则分别评估正确结果是否位列候选集的前五名和前三名。
![TablePilotEval2](/images/TablePilotEval.png)
（GPT4o好强....）
总体来说是SFT分析微调+SFT&DPO排序微调效果最好，4o的R@3比baseline高了25%，相比未调参的vanilla高了10%，效果非常明显。