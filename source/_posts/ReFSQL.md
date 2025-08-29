---
title: ReFSQL
date: 2025-08-26 09:14:47
tags:
---
**use sepcial RAG retreieve similar sqls for prompts, combining both supervised and unsupervised learning, to solve the gap between"specific knowledge and general knowledge."**
**from"ReFSQL: A Retrieval-Augmentation Framework for Text-to-SQL  Generation“**
## Overall
- attributes the errors in nl2sql:
  ```
  the underestimation of the inherent structural characteristics 
  as well as the gap between specific knowledge and general knowledge.
  ```
  and also:
  ```
  SQL language possesses unique inherent structural characteristics, 
  i.e., heavily relies on the database schema and SQL grammar but lacks 
  robust natural language attributes. Previous methods that directly
  align the distributions of natural language and SQL query may fail 
  to capture these inherent structural characteristics in SQL .
  ```
- propose a retrievalargument framework, namely ReFSQL
- structure-enhanced retriever  and the generator. Structure-enhanced retriever  is designed to identify samples with comparable specific knowledge in an unsupervised way.
- incorporate the retrieved samples’ SQL into the input, enabling our model to acquire prior knowledge of similar SQL grammar.
- mahalanobis contrastive learning method, facilitates the transfer of the sample toward the  specific knowledge distribution constructed by  the retrieved samples.
## Methods
![ReFSQL](/images/ReFSQL.png)
### 0) Problem setup (notation)

Each training example is a natural-language **question** $Q$, a **database schema** $S=(T,C)$, and its gold **SQL** $Y$. The goal is to generate SQL from $(Q,S)$. The framework has two big parts: a **structure-enhanced retriever** and a **generator** (encoder–decoder LM). See Figure 2 for the overview.

---

### 1) Structure-enhanced retriever (two stages)

#### 1A) SQL-Structure-enhanced **Question** Retriever (SQLSE) — Sec. 3.1.1

**Why:** Learn a question embedding space where questions with *similar SQL structure* are close, even if they’re phrased differently.

**How it’s built (training time):**

1. Convert each gold SQL to a **tree**; compute **Tree-Edit Distance** between SQL trees to quantify structural similarity:

$$
\delta_t = \mathrm{TED}(tr_1, tr_2) \tag{1}
$$

Use high-similarity pairs as **positives** and low-similarity pairs as **negatives**.&#x20;

2. Encode questions with a **frozen BERT**:

$$
h_{q_i}=\mathrm{Bert}(q_i) \tag{2}
$$

Train with an InfoNCE-style **contrastive loss** so anchors are closer to their TED-based positives than to negatives:

$$
\mathcal L_{sc} = -\log \frac{\exp(D_{\cos}(h_{q_i},h_{q_i}^+)/\tau)}{\sum_{q_j}\exp(D_{\cos}(h_{q_i},h_{q_j})/\tau)} \tag{3}
$$

After this, retrieve **top-m** similar questions for each query by cosine in the learned space.&#x20;

> Intuition: TED supplies **which pairs should be close/far**; the question encoder learns an embedding space that reflects SQL-structure similarity. TED itself isn’t used at inference—only to create contrastive pairs during training.&#x20;

#### 1B) Linking-Structure-enhanced **Schema** Retriever (LSES) — Sec. 3.1.2

**Why:** Among those m question-neighbors, keep the ones whose **schema interactions** look most like the current case.

**How it’s built:**

1. Build a heterogeneous **interaction graph** $g_i$ that mixes:

* **Schema-encoding** edges (e.g., **BELONGS-TO** for column→table; **FOREIGN-KEY** across tables).
* **Schema-linking** edges between question tokens and schema items using n-gram matches, with **exact vs partial** matches to reduce noise.&#x20;

2. Encode the graph with **R-GCN** to get an embedding:

$$
h_{g_i}=\mathrm{R\mbox{-}GCN}(g_i) \tag{4}
$$

Use cosine similarity on $h_{g}$ to **re-rank** the m candidates and keep the top-k as the most schema-compatible neighbors.&#x20;

---

### 2) Prompt construction (what the model sees) — end of Sec. 3.1.2

From those top-k neighbors, collect their gold SQLs and append them as a **simple SQL prompt**:

$$
\text{SP}_i=\text{“the similar SQL was:” } sql_{i1} \;|\; \cdots \;|\; sql_{ij}
$$

Then **serialize** the full input as:

$$
X_i = Q_i \;|\; S_i \;|\; t_1: c_{11},\dots \;|\; t_2:\dots \;|\; \text{SP}_i \tag{5}
$$

So the encoder sees the **question**, the **schema** (tables/columns), **and** a strip of **similar SQLs**.&#x20;

---

### 3) Generator (encoder–decoder LM) — Sec. 3.2

They instantiate the generator with **T5** (but note the framework is model-agnostic).

**Encoder:**

$$
h_{X_i}=\mathrm{T5\mbox{-}Encoder}(X_i) \tag{6}
$$

This bakes in the question, schema, and “similar SQL” prompt.&#x20;

**Decoder training:** two losses, combined.

##### 3A) Contrastive sample construction (for the decoder) — Sec. 3.2.2

From the **retrieved set** $M$ (the top-m from SQLSE), take:

* **Positives:** the **top-k** after LSES re-ranking.
* **Negatives:** lower-ranked items in $M$.

Compute the **encoder states** of those items and estimate **two Gaussians** in embedding space:

* Similar-sample distribution $f_\phi(h_{x^+}) \sim \mathcal N(\mu_+, \sigma_+^2)$ from positives,
* Dissimilar distribution $f_\phi(h_{x^-}) \sim \mathcal N(\mu_-, \sigma_-^2)$ from negatives.
  (This is just the sample mean/variance over the positive and negative neighbor embeddings.)&

##### 3B) Mahalanobis **contrastive** objective — Eq. (7)

Pull the current sample’s representation **toward** the similar distribution and **away** from the dissimilar one using **Mahalanobis distance**:

$$
\mathcal L_{MA} = -\log \frac{\exp\big(D_{ma}(f_\phi(h_{x^+_j}), h_{x_i})/\tau\big)}{\sum_{x_j\in M}\exp\big(D_{ma}(f_\phi(h_{x_j}),h_{x_i})/\tau\big)} \tag{7}
$$

with $D_{ma}$ computed vs the Gaussian $ \mathcal N(\mu,\sigma^2)$, implemented as $(h_{x_i}-\mu)^\top (\sigma^2 I)^{-1} (h_{x_i}-\mu)$. This teaches the decoder’s latent to **align with the “specific knowledge” distribution** defined by its most relevant neighbors.&#x20;

##### 3C) Standard token-level **MLE** — Eq. (8)

Usual cross-entropy over target SQL tokens:

$$
\mathcal L_{CE} = -\frac{1}{N}\sum_{i=1}^N \sum_{j=1}^{V} y_{i,j}\log \hat y_{i,j} \tag{8}
$$

**Total loss:** $\mathcal L = \mathcal L_{CE} + \mathcal L_{MA}$.

> Intuition: the **prompted neighbors** give the model explicit examples of SQL grammar; the **Mahalanobis loss** shapes the hidden representation so each example sits inside the *local* distribution of similar cases, not just the *global* language-model prior.&#x20;

---

### 4) Inference (test time)

For a new $(Q,S)$:

1. **SQLSE** embeds the question and retrieves **top-m** neighbors by cosine (no TED here—TED was only for creating training pairs).&#x20;
2. **LSES** builds the interaction graph, computes R-GCN embeddings, and **re-ranks** to keep **top-k** schema-compatible neighbors.&#x20;
3. Build **SP** from their SQLs and serialize **$X$** with $Q$, $S$, **SP** (Eq. 5).&#x20;
4. Feed $X$ to the **generator**; decode the SQL (beam search etc. per backbone).&#x20;

---

### 5) Why these pieces matter (evidence)

* The retriever and the Mahalanobis contrastive module each contribute measurable gains; removing either drops EM/EX in ablations (Table 3).&#x20;
* The approach is **backbone-agnostic** and helps from T5-small up through Flan-T5 (Table 2).&#x20;

---

#### TL;DR flow you can keep in mind

**Train:**
SQL→TED→(pos/neg question pairs)→train question embeddings→SQLSE retrieves m → LSES re-ranks to k → build SP → encode with T5 → decode with MLE + **Mahalanobis contrastive** using neighbor-based Gaussians.

**Test:**
SQLSE (m) → LSES (k) → SP prompt → T5 → SQL. No TED, no labels—just retrieval + generation.

## Suppluments，Details
- sql-tree: from "SyntaxSQLNet: Syntax Tree Networks for Complex and Cross-Domain Text-to-SQL Task"
  - ![sql_tree](/images/sql_tree.png)
- Tree-Edit-Distance(TED)
  - The tree edit distance between ordered labeled trees is the minimal-cost sequence of node edit operations that transforms one tree into another. We consider following three edit operations on labeled ordered trees:
    - delete a node and connect its children to its parent maintaining the order.
    - insert a node between an existing node and a subsequence of consecutive children of this node.
    - rename the label of a node.
  -There are many different sequences that transform one tree into another. We assign a cost to each edit operation. Then, the cost of an edit sequence is the sum of the costs of its edit operations. Tree edit distance is the sequence with the minimal cost
- the SQLse retriever uses a frozen Bert model with a additional trained prodection layer.
- The R-GCN refers to Relational Graph Convolutional Network, used to encode the heterogeneous graph of the schema.
