---
title: WizardLM and evol-instruct
date: 2025-05-30 17:55:08
tags:
---
**文献阅读：通过指令演化生成大规模复杂指令数据集，对模型进行监督微调**
**from"WizardLM: Empowering Large Language Models to Follow Complex Instructions"by Can Xu,Qingfeng Sun,Kai Zheng, Xiubo Geng,Pu Zhao,Jiazhan Feng,Chongyang Tao,Qingwei Lin,Daxin Jiang**
## 总览
* 大部分自然语言数据集中包含的指令有重叠，且类型少。
* instructGPT和ChatGPT的成功建立在人工指令数据集的基础上，证明了多样的指令提示词进行指令微调的巨大潜力。
* 人工构建大模型的指令数据集昂贵且指令复杂度趋低。
* 提出Evol-Instruct方法，LLM构建不同复杂度的多样指令。
* 随机选择 “深度进化”或“广度进化”来将简单指令升级为更复杂的指令或创建新指令以增加多样性。
* 深度进化包括五种操作：添加约束、深化、具体化、增加推理步骤和复杂化输入。
* 广度进化即变异，基于给定指令生成全新的指令。
* 由于进化后的指令由 LLM 生成，有时进化会失败，因此采用指令淘汰器过滤失败的指令“淘汰进化”。
* 选择 Alpaca的训练数据（从175条人工创建的种子指令生成）作为初始数据集，使用ChatGPT进行四轮进化，最终获得 25 万条指令。与Vicuna的7万条真实用户数据公平对比，从25万条数据中采样同等数量的数据，训练LLaMA 7B模型，命名为WizardLM。由于之前的指令遵循测试数据集中高难度指令比例较低，手动创建了一个新的难度平衡测试数据集Evol-Instruct。
![evolInstruct_example](/images/evolInstruct_example.png)
## 深度演化
深度进化通过五种类型的提示增强指令，使其更加复杂和困难：添加约束、深化、具体化、增加推理步骤和复杂化输入。要求大语言模型创建具有挑战性但合理的指令，而不是由人工智能任意想象的指令。
为避免指令集中充斥着极其复杂的指令（这会损害训练模型的泛化性能），难度需要逐步增加。让每次进化 “稍微难一点”，并限制最多添加 10 到 20 个单词。以下是五种演化的prompt模板。
* 添加约束：
```
I want you act as a Prompt Rewriter.
Your objective is to rewrite a given prompt into a more complex version to make those famous AI systems
(e.g., ChatGPT and GPT4) a bit harder to handle.
But the rewritten prompt must be reasonable and must be understood and responded by humans.
Your rewriting cannot omit the non-text parts such as the table and code in #Given Prompt#:. Also, please
do not omit the input in #Given Prompt#.
You SHOULD complicate the given prompt using the following method:
Please add one more constraints/requirements into #Given Prompt#
You should try your best not to make the #Rewritten Prompt# become verbose, #Rewritten Prompt# can only
add 10 to 20 words into #Given Prompt#.
‘#Given Prompt#’, ‘#Rewritten Prompt#’, ‘given prompt’ and ‘rewritten prompt’ are not allowed to appear in
#Rewritten Prompt#
#Given Prompt#:
<Here is instruction.>
#Rewritten Prompt#:
```
* 深化：
```
I want you act as a Prompt Rewriter.
Your objective is to rewrite a given prompt into a more complex version to make those famous AI systems
(e.g., ChatGPT and GPT4) a bit harder to handle.
But the rewritten prompt must be reasonable and must be understood and responded by humans.
Your rewriting cannot omit the non-text parts such as the table and code in #Given Prompt#:. Also, please
do not omit the input in #Given Prompt#.
You SHOULD complicate the given prompt using the following method:
If #Given Prompt# contains inquiries about certain issues, the depth and breadth of the inquiry can be
increased. or
You should try your best not to make the #Rewritten Prompt# become verbose, #Rewritten Prompt# can only
add 10 to 20 words into #Given Prompt#.
‘#Given Prompt#’, ‘#Rewritten Prompt#’, ‘given prompt’ and ‘rewritten prompt’ are not allowed to appear in
#Rewritten Prompt#
#Given Prompt#:
<Here is instruction.>
#Rewritten Prompt#:
```
* 具体化：
```
I want you act as a Prompt Rewriter.
Your objective is to rewrite a given prompt into a more complex version to make those famous AI systems
(e.g., ChatGPT and GPT4) a bit harder to handle.
But the rewritten prompt must be reasonable and must be understood and responded by humans.
Your rewriting cannot omit the non-text parts such as the table and code in #Given Prompt#:. Also, please
do not omit the input in #Given Prompt#.
You SHOULD complicate the given prompt using the following method:
Please replace general concepts with more specific concepts. or
You should try your best not to make the #Rewritten Prompt# become verbose, #Rewritten Prompt# can only
add 10 to 20 words into #Given Prompt#.
‘#Given Prompt#’, ‘#Rewritten Prompt#’, ‘given prompt’ and ‘rewritten prompt’ are not allowed to appear in
#Rewritten Prompt#
#Given Prompt#:
<Here is instruction.>
#Rewritten Prompt#:
```
* 增加推理步骤：
```
I want you act as a Prompt Rewriter.
Your objective is to rewrite a given prompt into a more complex version to make those famous AI systems
(e.g., ChatGPT and GPT4) a bit harder to handle.
But the rewritten prompt must be reasonable and must be understood and responded by humans.
Your rewriting cannot omit the non-text parts such as the table and code in #Given Prompt#:. Also, please
do not omit the input in #Given Prompt#.
You SHOULD complicate the given prompt using the following method:
If #Given Prompt# can be solved with just a few simple thinking processes, you can rewrite it to
explicitly request multiple-step reasoning.
You should try your best not to make the #Rewritten Prompt# become verbose, #Rewritten Prompt# can only
add 10 to 20 words into #Given Prompt#.
‘#Given Prompt#’, ‘#Rewritten Prompt#’, ‘given prompt’ and ‘rewritten prompt’ are not allowed to appear in
#Rewritten Prompt#
#Given Prompt#:
<Here is instruction.>
#Rewritten Prompt#:
```
* 复杂化输入(不唯一)：
```
I want you act as a Prompt Rewriter. Your objective is to rewrite a given
prompt into a more complex version using dataformat to make those famous
AI systems (e.g., chatgpt and GPT4) more difficult to handle. But the
rewritten prompt must be reasonable and must be understood and responded
by humans.
You must add [python code] format text as input data in [Rewritten Prompt]
The Given Prompt:
Transformat python code
Rewritten Prompt(MUST contain a specific python code as input):
I have the following Python code:
cursor . execute (" INSERT INTO table VALUES var1 , var2 , var3 ,")
where var1 is an integer, var2 and var3 are strings.
How can I write the variable names without Python including them as part of
the query text?
####
```

## 广度演化
广度演化旨在提升主题覆盖范围、技能覆盖范围和整体数据集的多样性。开放域指令微调数据集通常规模较小，缺乏主题和技能的多样性。为解决这一问题，设计了一种基于给定指令生成全新指令的提示，要求新指令更具长尾性。广度进化prompt模板如下：
```
I want you act as a Prompt Creator.
Your goal is to draw inspiration from the #Given Prompt# to create a brand new prompt.
This new prompt should belong to the same domain as the #Given Prompt# but be even more rare.
The LENGTH and difficulty level of the #Created Prompt# should be similar to that of the #Given Prompt#.
The #Created Prompt# must be reasonable and must be understood and responded by humans.
‘#Given Prompt#’, ‘#Created Prompt#’, ‘given prompt’ and ‘created prompt’ are not allowed to appear in
#Created Prompt#.
#Given Prompt#:
<Here is instruction.>
#Created Prompt#:
```

模型同时也要生成进化后指令的回应，以$<Here is instruction.>$分隔。

## 淘汰进化：
将以下四种情况归类为指令进化失败：

* 进化后的指令相比原始指令未提供任何信息增益。使用ChatGPT进行判断，具体细节请参考附录 G。
* 进化后的指令使 LLM 难以生成响应。当生成的响应包含 “sorry” 且长度较短（即少于 80 词）时，通常表明 LLM 难以处理该进化指令，因此可基于此规则进行判断。
* LLM 生成的响应仅包含标点符号和停用词。
* 进化后的指令明显复制了进化提示中的部分词汇，如 “given prompt”“rewritten prompt”“#Rewritten Prompt#” 等。

## 实验细节
* 训练数据集构建：
  以Alpaca的5.2万条指令数据集为初始数据，通过迭代执行M=4轮进化，最终获得了25万条指令。在每轮进化中，针对每条指令，**以相等的概率**从总共6种提示（5种深度进化提示和1种广度进化提示）中随机选择一种进行进化。通过Azure OpenAI ChatGPT API执行，最终也利用ChatGPT生成响应。生成响应时使用温度参数1，最大生成token数设置为2048，并将频率惩罚设为0，top-p设为0.9。构建完整数据集共调用API 52×4×3=62.4万次。
* 测试集构建：
  Evol-Instruct测试集包含来自在线开源项目、平台和论坛等多种来源的**真实人类指令**。通过对数据的分析，确定了29种不同的技能，涵盖编程生成与调试、数学、推理、复杂格式处理、写作、专业学科等人类主要需求领域。下图展示了测试集中实例和技能的分布情况，该测试集包含218个实例，每个实例对应一种特定技能的指令。
  与Vicuna用于评估指令遵循模型的基准测试集相比，其仅包含80个实例和9种技能，规模更小且多样性更低。下图显示了测试数据在不同实例间的难度和复杂度分布：Evol-Instruct的测试数据分布更为均匀，涵盖不同难度和复杂度级别的指令；而Vicuna和Alpaca的数据集分布存在偏差，主要包含低难度和低复杂度的指令，表明这两种语料库难以应对更复杂、更高要求的评估场景。
  ![evolInstruct_testSet](/images/evolInstruct_testSet.png)
  难度评估的prompt模板如下：
```
We would like you to evaluate and rate the difficulty and complexity of the following question. You
should give an overall score on a scale of 1 to 10, where a higher score indicates higher difficulty and
complexity. You must just give a score without any other reasons.
## Question:
< Here is instruction. >
## Score:
```
* 模型训练：
  使用预训练的 LLaMA 7B 模型进行初始化，采用Adam优化器，初始学习率为2×10⁻⁵，最大token数为2048，每个GPU的batch大小为8。在8块V100GPU上使用Deepspeed Zero-3训练3个epoch，耗时70小时。为确保公平对比，将Alpaca原始的Davinci-003响应替换为ChatGPT的响应，并从数据中采样7万条指令子集用于训练WizardLM。
* 推理设置：
  为减少输出随机性并确保输出更集中和确定，WizardLM和基线模型在推理时均将温度设为1，top-p设为0.9，使用束搜索大小为1，最大生成长度设为2048。
* 人工评估：
  为评估 WizardLM，在Evol-Instruct测试集上进行了人工评估，对WizardLM与基线模型进行了盲法两两对比。招募了10名高学历注释员，向每位注释员展示Alpaca、Vicuna-7B、WizardLM和ChatGPT的响应（随机打乱以隐藏来源），要求其根据固定标准判断哪一响应更优，并对四个响应从1到5进行排名（1 表示最佳），允许对相当的实例给出相同分数。为估算胜率，比较每对模型之间的胜负和平局频率。
  人工评判标准如下：
```
The annotators then judge which response is better from five aspects:
(1) Relevance: Assessing the model’s ability to correctly interpret the semantic meaning of the context
and questions.
(2) Knowledgeable: Whether the model can accurately use various and detailed knowledge for
problem-solving.
(3) Reasoning: Assessing the model’s ability to execute correct reasoning processes or devise valid
reasoning concepts to solve problems.
(4) Calculation: Evaluating whether the model can perform accurate mathematical computations of
the provided formulas in the domains of math, biology, chemistry and physics.
(5) Accuracy: Evaluating whether the model can perform correctly in the corresponding for a given
instruction.
```
人工评估结果：
![evolInstruct_humaneval](/images/evolInstruct_humaneval.png)
* GPT-4自动评估
采用Vicuna提出的基于GPT-4的自动评估框架来评估聊天机器人模型的性能。遵循与Vicuna相同的GPT-4超参数、提示设置和评估方法。为了缓解顺序偏差，在两两比较中交替WizardLM和其他模型的位置：奇数ID时WizardLM为第一个，偶数ID时为第二个。WizardLM在Evol-Instruct测试集上大幅优于Alpaca-7B和Vicuna-7B（分别比Alpaca-7B和Vicuna-7B高6.2%和5.8%），在Vicuna测试集上与Vicuna-7B性能相当。
![evolInstruct_GptEval](/images/evolInstruct_GptEval.png) 
**不同技能的表现**。比较WizardLM和ChatGPT在Evol-Instruct测试集上的技能水平。结果表明，WizardLM平均达到ChatGPT性能的78%，在17项技能上几乎超过90%。然而，WizardLM在代码、数学和推理场景中表现不佳，与ChatGPT存在明显差距。  
![evolInstruct_GptComparision](/images/evolInstruct_GptComparision.png)

尽管人工评估认为WizardLM由于ChatGPT，但WizardLM在困难技能上输给了ChatGPT，这与上述人工评估的结论相反。主要原因在于：人类更偏好整洁生动的格式，且人们更倾向于为可编译通过的代码或数学问题额外加分。

考虑到LLama 7B和chatGPT参数量的差距，这种结果已经显示出evol-instrct方法的巨大潜力。
