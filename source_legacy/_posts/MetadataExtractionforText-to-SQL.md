---
title: Metadata Extraction for Text-to-SQL and LSH
date: 2025-07-09 09:43:57
tags:
---
**文献阅读：自动 sql filed description 提取，用以增强text2sql**
**from"Automatic Metadata Extraction for Text-to-SQL"**
## Overview
```
the most difficult part of writing correct SQL queries is
understanding what is in the database to begin with
```
LLMs are good at writing sql querys, but can make mistakes for the lack of comprehension of the data's meaning.
Examples of difficulties include:
- Lack of documentation: In a large number of cases,the database lacks documentation – no field or table descriptions. The new database user has only cryptic field and table names for guidance. 
- Incomplete or dated documentation: even with documentation, the content can be inaccurate due to the update of the databse itself.
- Unclear data formats:even with the documentation, the format of the data field can still be unclear.
- Multiple data formats: the data in fields can even be in different formats.
- Default valus: unwanted default valus should be exclued while processing options like join, select to reduce time consumption.

The system this paper gives can be concluded as:
- make a profile base on statistics data of each field.
- use the profile and simple description data to let LLM generate a detailed NL profile for every field.
- apply LSH to every filed'sdata, combined with the NL profile above to build schema for every table.
- when a question is given,first use a rather small sub-schema(for the whole schema can be too long to input LLM) for LLM to generate a sql query.
- see the sql query's resemblance to the field's schema data.if there's other fields's schema resemble with the question, but this field is not in the sql query, make LLM generate again with refined prompt, adding the fields in it.
- the step above can go k rounds.
- use the ultimate set of fields(schema) to generte the final sql.
## Profiling
### 1.troditional profiling
set statistics points like:
- The number of records in a table.
- For a field, the number of NULL vs. non-NULL values.
- For a field, the number of distinct values.
- For a field, the “shape” of a field, e.g. min and max,
number of characters (digits), alphabet
(upper/lower/punctuation/…), common prefixes, etc.
- For each field, a sample of the top-k field values.
- For each field, a minhash sketch. 
where minihash is used to rate the similarity of two fields：
> A **minhash sketch** is a collection of $K$ values computed by:
>
> $$
> mi(f) = \min(h_i(v_j) \mid \text{over all values } v_j \text{ of field } f)
> $$
>
> for $i$ ranging from 1 to $K$, and each $h_i$ is a different hash function.
>The **resemblance** between two fields $f$ and $g$ (denoted as $\text{res}(f, g)$) is approximated by:
>
>$$
>\text{res}(f, g) = \frac{1}{K} \sum_{i=1}^K \text{ if }(mi(f) = mi(g), 1, 0)
>$$
>This allows fast approximation of **Jaccard similarity** between sets of field values.

then,using mechanically generated English language for a description,such as:
```
Column CDSCode has 0 NULL values out of 9986 records.
There are 9986 distinct values. The minimum value is
'01100170109835' and the maximum value is
'58727695838305'. Most common non-NULL column values
are '01100170109835', '01100170112607', '01100170118489',
'01100170123968', '01100170124172', '01100170125567',
'01100170130401', '01100170130419', '01100176001788',
'01100176002000'. The values are always 14 characters long.
Every column value looks like a number. 
```
### 2.LLM for detailed NL profile summary: 
combining the info above, with more info like the description csv Bird has, the table name and the names of other fields in the table,LLM can do two things:
- explain the short names of the field, for example:
    ```
    The CDSCode column stores unique 14-character numeric
    identifiers for each school in the database, where CDS stands
    for County-District-School. 
    ```
- tell in which fomat the data is stored:
    Take an example, int the origin csv description Bird gives only shows:
    ```
    A list of formats the card is legal to be a commander in 
    ```
    while LLM can extend it with all the info above:
    ```
    The leadershipSkills column stores JSON-formatted data
    indicating the formats in which a card is legal to be used as a
    commander, such as Brawl, Commander, and Oathbreaker 
    ```
## Schema linking
although LLM can generate detailed descriptions for every field, in the next steps to come, the LLM can hardly take in all these info:
- overflow the token limit for LLM systems
- LLM will pick up on the material in the beginning and the end, but tend to ignore the part in the middle
these sympotms are observed in [TPCM+24]

**Schema linking is used to identifying which fields are relevant to generating an SQL query in response to a question.**
when generating profile summary, two types of summary, long and short, are generated at the same time. Short summary are used for schema linking, Longer ones are for text2sql jobs to come.


### **Schema-Based SQL Query Generation and Augmentation Procedure**

For each of the following five cases of schema:

* **a)** Focused schema, minimal profile
* **b)** Focused schema, maximal profile
* **c)** Full schema, minimal profile
* **d)** Full schema, maximal profile
* **e)** Focused schema, full profile

Perform the following steps:


#### **1. SQL Generation**

* **a.** Use the LLM to generate an SQL query `Q` based on the provided **Question** and **Schema**.

#### **2. Extract Fields and Literals**

* **b.** Let `Fields_Q` be the set of fields referenced in `Q`.
* **c.** Let `Lits_Q` be the set of literals in `Q`.
* **d.** Initialize:

  * `LitFields_Q = []` (fields associated with literals not in `Fields_Q`)
  * `MissingLits = []` (literals not explained by schema fields)

#### **3. Literal-Field Mapping**

For each literal `l` in `Lits_Q`:

* **i.** Use LSH indices in the profile to identify fields `Fields_l` which contain `l` as a value.
* **ii.** If none of the fields `f` in `Fields_l` are in `Fields_Q`:

  * 1. Add `Fields_l` to `LitFields_Q`
  * 2. Add `l` to `MissingLits`

#### **4. Augmentation and Retry**

If `LitFields_Q` is not empty **and** the number of retries < `MaxRetry`:

* **i.** Create `AugmentedSchema` by adding any fields in `LitFields_Q` not already present in `Schema`.
* **ii.** Write a **new prompt** for the LLM asking it to revise SQL query `Q`, suggesting use of fields which contain literals in `MissingLits`.
* **iii.** Repeat steps **2.b through 2.e** using the revised query.

#### **5. Accumulate Context**

* Add `Fields_Q` and `Lits_Q` to global `Fields` and `Lits`.

### **Final Output**

* Return `Fields_Q` as the final set of fields used for providing context.

## result
![Metadata_extraction_res](/images/Metadata_extraction_res.png)


# local sensitive hash(LSH)


## Full Pipeline Overview

```
Raw Text → Shingles → Sets → MinHash Signatures → LSH (Banding + Bucketing) → Candidate Pairs → Exact Jaccard Check
```


## Step 1: Shingling

### What it does:

Breaks a document into overlapping substrings of length `k` (called **k-shingles**).

### Example:

For the string `"abcde"` and `k=3`, the shingles are:

```
["abc", "bcd", "cde"]
```

The output of this step is:
**Each document = a set of shingles (strings)**


## tep 2: Convert to Sets with Global IDs

Assign a unique integer to every shingle that appears in your entire dataset:

Example:

```
"abc" → 0  
"bcd" → 1  
"cde" → 2  
...
```

So a document becomes:

```
Doc1 = {"abc", "bcd", "cde"} → {0, 1, 2}
```


## Step 3: MinHash Signatures

### Purpose:

Create a **compressed signature vector** that preserves Jaccard similarity.

### How:

1. Generate `m` random hash functions like:

$$
h_i(x) = (a_i \cdot x + b_i) \mod p
$$

2. For each document's set, compute:

$$
\text{MinHash}_i(\text{Doc}) = \min \{h_i(x) \text{ for all } x \in \text{Doc}\}
$$

3. After doing this `m` times, we get a **signature vector** of length `m` per document.

### Example:

For `m = 6`, Doc1’s signature might look like:

```
[5, 12, 3, 17, 0, 8]
```


## Step 4: LSH (Locality-Sensitive Hashing)

### Purpose:

Efficiently find **pairs of documents with similar MinHash signatures**, avoiding all pairwise comparisons.

### The Idea:

Documents with **similar MinHash signatures** will likely **share identical values in some parts (bands)**.


### Banding Technique:

say:

* Signature length = 12 (i.e., m = 12)
* Split into `b = 3` bands, each of `r = 4` rows

Now a signature like:

```
[5, 12, 3, 17, 0, 8, 9, 7, 14, 4, 1, 11]
```

gets divided into:

* Band 1: `[5, 12, 3, 17]`
* Band 2: `[0, 8, 9, 7]`
* Band 3: `[14, 4, 1, 11]`

### Hashing:

Each band (which is a 4-tuple) is hashed into a **bucket**.

**Key Rule:**

> If two documents hash to the same bucket in any band, they are considered a **candidate pair**.

### Example:

* Doc A and Doc B have the **same Band 2**: `[0, 8, 9, 7]`
* So they get hashed to the same bucket for Band 2 → Candidate Pair 

This greatly reduces the number of comparisons.


## Step 5: Exact Jaccard Similarity Check

### Purpose:

LSH gives us **candidate pairs**; now we check which ones are **truly similar**.

### How:

Go back to the **original shingle sets** and compute:

$$
Jaccard(A, B) = \frac{|A \cap B|}{|A \cup B|}
$$

### Example:

* Doc1 shingles = {"abc", "bcd", "cde"}
* Doc2 shingles = {"bcd", "cde", "def"}

$$
Jaccard = \frac{2}{4} = 0.5
$$

If your threshold is 0.8 → Not a match 
If your threshold is 0.4 → It’s a match 