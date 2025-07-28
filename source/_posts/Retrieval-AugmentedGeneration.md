---
title: Retrieval-AugmentedGeneration
date: 2025-07-28 08:53:45
tags:
---
**RAG**
**from"Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks“**
## Key Points
- Marginalization
  - Instead of picking one document and hoping it's right, average (or sum) over all possibilities, weighted by how likely each one is.
  - **marginalizing over the latent variable** `z`:
    $$
    p(y|x) = \sum_z p(z|x) \cdot p(y|x, z)
    $$
    * `x` is the input (e.g., question).
    * `z` is a retrieved document (latent variable).
    * `y` is the output (e.g., answer).
    * `p(z|x)` is how likely document `z` is given the query.
    * `p(y|x, z)` is how likely the output is given the input and document.
- parametric memory and non-parametric memory
  - parametric memory models like GPT, BERT, and BART store all their knowledge in their weights, used here as generator.
  - refers to external knowledge sources that the model can access dynamically during inference.
  - in DPR(Dense Passage Retriever), the model only has two encoder for ducoments and querys, no corss-attention,MLP,decoder. So it doesn't have a parametric memory.
- Maximum Inner Product Search (MIPS)
  - example:
      * Wikipedia is split into \~21 million disjoint 100-word passages.
      * Each passage is encoded into one single dense vector using a document encoder (e.g., 768-dimensional for BERT-base).
      * These vectors are stored in an index built with **FAISS** (a library for fast similarity search).
      * Queries are also encoded into dense vectors using a separate BERT-based query encoder.
      * Retrieval is performed using **Maximum Inner Product Search (MIPS)** to find the top-k documents most similar to the query vector.
   - MIPS is actually calculating the similarity between word embeddings.
## Methods
- ![RAG](/images/RAG.png)

### Shared Structure

Both models consist of:

* A **retriever**: finds the top-*k* relevant documents (passages) given an input (query).
* A **generator**: generates an output sequence (like an answer or sentence) conditioned on the input and the retrieved documents.
* A **probabilistic formulation**: treats the retrieved documents as **latent variables** and marginalizes over them.
### RAG-Sequence Model
each **entire output sequence** is generated using the **same retrieved document**, each document has a top series.
* Retrieve top-*k* documents `z₁, z₂, ..., zₖ` for input `x`.
* For each document `zᵢ`, compute the probability of generating output `y`:

  $$
  p(y | x, zᵢ)
  $$
* Final output probability is the **weighted sum** over all documents:

  $$
  p(y | x) = \sum_{z \in \text{top-k}} p(z|x) \cdot p(y|x, z)
  $$
* One **document is responsible** for the whole sequence during decoding.  
### RAG-Token Model
Each **token** in the output can be generated using a **different retrieved document**. All the documents have only one top output series.
* For each output token `yᵢ`, compute:

  $$
  p(yᵢ | x, y₁:i₋₁) = \sum_{z \in \text{top-k}} p(z|x) \cdot p(yᵢ | x, z, y₁:i₋₁)
  $$
* This means the model dynamically chooses **which document** to use for each token.
* More flexible: can pull from different documents while generating.
