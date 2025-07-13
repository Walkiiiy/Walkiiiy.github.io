---
title: CHASE-SQL:Multi-CandidateGenerateSelectForText-to-SQL
date: 2025-07-10 18:29:25
tags:
---
**文献阅读：多种途径生成sql并选优，用以增强text2sql**
**from"CHASE-SQL: Multi-Path Reasoning and Preference Optimized Candidate Selection in Text-to-SQL"**
## Overview
```
In the broader code
generation domain, utilizing LLMs to generate a wide range of diverse candidates and select the best one has
proven to be effective.
```

before generating sqls, use LSH value retrival for a selected set of rows:
```
retrieve the most syntactically-similar words, and re-rank them based on 
embedding-based similarity and edit distance
```
then use three main approaches of generating both diversified and high-qulity sql:
-  divide-and-conquer
-  chain-of-thought
-  help the model better understand the underlying data schema 
finally use a LSH value retrieval method to select the  after these approaches generates different sqls, use a scoring agent to select the best(apart from troditional self-consistency method)
![CHASEsql](/images/CHASEsql.png)
## Value retrival
value retrival is first proposed by team in Stanford University, in "CHESS: Contextual Harnessing for Efficient SQL Synthesis", which achieved 71% in Bird test set in 2024. At the time this artical is finished, the top researches in bird rank list all used methods like Chess for a shorten sql schema input.
```
For each keyword, we employ locality-sensitive hashing (LSH) (Datar
et al., 2004) to retrieve the most syntactically-similar words, and re-rank them based on embedding-based
similarity and edit distance. This approach is robust to typos in the question and considers keyword semantics
during retrieval.
```
## Divide and Conquer CoT
decomposite the quesition into subquestions, conquer them one by one then assemble into the final sql.This approach is good at solving complex questions involving nested queries, e.g. intricate WHERE or HAVING conditions, and queries requiring advanced mathematical operations.
Algorithm described in the paper was:
```
Input: Set of human-annotated few-shot examples M, user question Qu, target database D associated with the
question, and a large language model (LLM) θ.
Divide:
1: Sq ← θ(M, D, Qu) // Decompose the original question Qu into a set of sub-questions Sq
2: Ssql ← ∅ // Initialize an empty set Ssql to store partial SQL queries for each sub-question
Conquer:
3: for each sub-question qi in Sq do
4: // Generate a partial SQL query for each sub-question qi
5: Ssql ← Ssql ∪ {θ(M, D, Qu, q1, ..., qi, sql1, ..., sqli−1)}
6: end for
Assemble:
7: Sf ← θ(M, D, Qu, Sq, Ssql) // Assemble the final SQL query Sf from all sub-queries in Ssql
8: return Sf
```
an classical example of a complex question can only be solved by divide and conquer Cot is:
![CHASE_divideConquer](/images/CHASE_divideConquer.png)
## Query Plan CoT
use CoT and Few-shot prompting strategy to make LLM mimics how a database engine internally executes a SQL query.
- use EXPLAIN command on the example sql
- the original sql EXPLAIN output is hard to understand, converted into a simplified, human-readable format.
- This simplified format is used in prompts to guide the LLM to reason step-by-step just like a query engine.

While the divide-and-conquer approach is better suited for decomposing complex questions, the query plan approach excels when questions require more reasoning over the relationships between different parts of the question and the database schema.
## Online Synthetic Example Generation
Generating tailored few-shot examples specifically for each test question, instead of using static, pre-defined examples.
first, the LLM generates examples demonstrating diverse SQL features, such as:
- GROUP BY, HAVING, ORDER BY, JOIN
- Aggregation: SUM, COUNT, AVG
- Nested queries

then, LLM generates examples using filtered relevant columns, guided by rules to ensure examples are schema-relevant.
then incorporating these examples, follow the query distribution in Bird dataset.
## Query Fixer
Using fixed prompts in LLM to fix the synatex error in generated sql querys, making sure the query is valid(not necessarily correct).
## Selection Agent
first, use SFT to train a selection agent:
```
generate candidate SQL queries on the training set (of Text-to-SQL benchmarks), and group them into clusters based on their execution results. For cases where at least one cluster contains correct queries and others contains incorrect ones, we create training examples in the form of tuples (Qu, Ci, Cj, Dij, yij), where Qu is the user’s question, Ci and Cj are the two candidate queries being compared, Dij is the database schema used by both candidates, and yij ∈ 0, 1 is the label indicating whether Ci or Cj is the correct query.
```
then compare every query pair with algorithm:
```
Input: 
- Set of candidate SQL queries C = {c1, c2, ..., cn}
- User question Qu
- Hint Hu
- Target database D
- Selection model θp
- er(ci, D): execution result of ci on D

1: Initialize score ri = 0 for each candidate ci in C

2: For each distinct pair (ci, cj) where i ≠ j:
    3: If er(ci, D) == er(cj, D): 
        4: Mark ci as the winner (w = i)
    5: Else:
        6: Si,j ← schema_union(ci, cj, D) // Combine schemas used by ci and cj
        7: w ← θp(Si,j, Qu, Hu, ci, cj) // Selection model chooses winner between ci and cj
    8: Increment the score of the winner: rw = rw + 1

9: End For

10: Select cf ← argmax ri over all ci in C // Candidate with highest score

11: Return cf as the final SQL query
```
## Result

single method:
| **Method**                       | **Execution Accuracy (%)** | **With Query Fixer (%)** |
| -------------------------------- | -------------------------- | ------------------------ |
| **Baseline (Zero-shot CoT)**     | 57.75                      | 61.58                    |
| **Query Plan CoT**               | 63.62                      | 65.51                    |
| **Divide-and-Conquer CoT**       | 63.92                      | 65.77                    |
| **Online Synthetic Example CoT** | **67.09**                  | **68.02**                |


combined and ablation expriments:
| **Method**                        | **Execution Accuracy (%)** | **Δ (%)** |
| --------------------------------- | -------------------------- | --------- |
| **CHASE-SQL (All)**               | 73.01                      | -         |
| CHASE-SQL w/ self-consistency     | 68.84                      | -4.17     |
| CHASE-SQL w/ ranker agent         | 65.51                      | -7.5      |
| CHASE-SQL w/o LSH                 | 70.09                      | -2.92     |
| CHASE-SQL w/o Query Fixer         | 69.23                      | -3.78     |
| CHASE-SQL w/o QP (Query Plan CoT) | 72.36                      | -0.65     |
| CHASE-SQL w/o OS (Synthetic CoT)  | 72.16                      | -0.85     |
| CHASE-SQL w/o DC (Divide-Conquer) | 71.77                      | -1.24     |

where w/o refers to without, Δ means persentage drop compared to full CHASE-SQL.