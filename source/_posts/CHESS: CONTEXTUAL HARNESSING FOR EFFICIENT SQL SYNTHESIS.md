---
title: CHESS:CONTEXTUAL HARNESSING FOR EFFICIENT SQL SYNTHESIS
date: 2025-07-29 09:39:00
tags:
---
**original LSH schema-linking paper**
**from"CHESS: CONTEXTUAL HARNESSING FOR EFFICIENT SQL SYNTHESIS"**
## Background
- Even some of the largest proprietary models, such as GPT-4, lag significantly behind human performance on text-to-SQL benchmarks, with a notable accuracy gap of 40% 
- main reason: caused by the need to effectively retrieve and integrate multiple sources of information
- challenges in text2sql:
    - users’ questions might not directly match the stored values in the database(left)
    - real-world database schemas often contain ambiguous column names, table names, and messy data(middle)
    - some questions may be inherently ambiguous, leading large language models (LLMs) to make subtle mistakes when generating SQL queries(right)
![Chess_challenges](/images/Chess_challenges.png) 
## Framework
using four agents:
- the Information Retriever (IR) extracts relevant data
- the Schema Selector (SS) prunes large schemas
- the Candidate Generator (CG) generates high-quality candidates and refines queries iteratively
- the Unit Tester (UT) validates queries through LLM-based natural language unit tests

the superiority of CHESS framework:
```
CHESS achieves 71.10% accuracy on the BIRD test set, within 2% of the leading proprietary method, while requiring approximately 83% fewer LLM calls.
```

## Methods
![Chess_framework](/images/Chess_framework.png)
#### Information Retriever (IR)
- Fetch relevant information like value and column's full names, descriptions.
- tools:
  - extract keywords: using few-shot LLM to extract the key words, phrases in from the input question.
  - **retrieve entity(hierarchical)**:
    1. **Retrieve Entities Using LSH**:
     * **Locality Sensitive Hashing (LSH)** is employed in the **pre-processing stage** to **index database values** based on their similarity.
    2. **Refine Results with Embedding Similarity**:
     * After retrieving the top-k values using LSH, **semantic similarity** is assessed using **embedding-based methods**.
     * The **embeddings** (vector representations of database values and keywords) are used to measure the **semantic distance** between the query and the database values. This ensures that the results retrieved via LSH are **conceptually aligned** with the user’s query.
    3. **Edit Distance**:
     * Paralleled with embedding-based medthods, the **edit distance** is calculated between the extracted keywords and the candidate values from the database. **Edit distance** measures **syntactic similarity** and helps identify the best matches that might have minor typographical differences or alternative expressions.
  - retrieve context: retrieve the most relevant metadata description, using embedding similarity.
#### Schema Selector (SS)
- Select necessary tables and columns required for generating the SQL query.
- tools: 
  - filter column: use LLM filter irrelevant columns.
  - select columns: select relevant columns.
  - select tables: select relevant tables.
#### Candidate Generator (CG)
- Generating candidates, excute, identify errors, revise.
- tools:
  - generate candidate query
  - revise
#### Unit Tester (UT)
- Selecting the most accurate SQL query from the pool of candidates.
- tools:
  - generate unit test:generate k unit tests, prompt is carefully constructed to produce high-quality unit tests that highlight the semantic differences between the candidate queries.
  - evaluate: takes multiple candidate queries and a single unit test as input, prompting an LLM to reason through each candidate and assigning a score.

## PS:
- when schema sizes are relatively small and the available LLMs are highly capable, the inclusion of the Schema Selector (SS) agent may introduce a precision-recall trade-off that can negatively impact accuracy. 
- how the Information Retriever and Schema Selector work together as schema-lingking:
   If the user asks, "What is the average math score for students?" the IR agent might retrieve the column “MathScore” and associated column descriptions. The Schema Selector then uses this information to select the correct column and table for the SQL query. 