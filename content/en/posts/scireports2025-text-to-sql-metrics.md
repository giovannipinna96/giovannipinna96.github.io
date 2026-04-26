---
title: "Redefining Text-to-SQL Metrics: Beyond Binary Evaluation"
date: 2025-07-02
draft: false
tags: ["Text-to-SQL", "Evaluation Metrics", "Semantic Similarity", "SQL", "LLM"]
categories: ["Research"]
description: "Introducing the Query Accuracy Score (QAS), a continuous evaluation metric for text-to-SQL systems that captures the full spectrum between 'perfectly correct' and 'completely wrong' by combining semantic and structural similarity."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
The dominant evaluation metrics for text-to-SQL systems — Exact Match and Execution Accuracy — are both binary, scoring each generated query as either fully correct or completely wrong. This erases critical distinctions between queries that are nearly correct and those that are fundamentally flawed. We introduce the Query Accuracy Score (QAS), a continuous metric that combines semantic similarity (using UAE-Code-Large-V1 code embeddings and cosine distance) with table similarity (using edit-distance comparison of query result tables). Evaluated on 11 text-to-SQL models across the BIRD benchmark, QAS reveals hidden quality differences invisible to binary metrics: models with similar execution accuracy turn out to have strikingly different error profiles. The two-component structure also enables differential diagnosis — distinguishing between intent failures (wrong query structure) and execution failures (right structure, wrong values). Published in Scientific Reports, 2025.
{{< /summary-box >}}

## Introduction

Text-to-SQL — the task of automatically translating natural language questions into SQL database queries — has seen tremendous progress thanks to Large Language Models. Systems built on GPT-4, specialized fine-tuned models, and various open-source alternatives can now handle increasingly complex queries across diverse database schemas. The promise is transformative: enabling anyone to query databases without SQL expertise.

But progress in any field is only as reliable as the metrics used to measure it. And the text-to-SQL field has a measurement problem. The two dominant evaluation metrics — **Exact Match (EM)** and **Execution Accuracy (EX)** — are both binary: they score each generated query as either completely correct (1) or completely wrong (0). This binary nature creates a substantial blind spot that can mislead both researchers and practitioners.

This paper, published in **Scientific Reports (2025)**, introduces the **Query Accuracy Score (QAS)** — a continuous metric that captures the rich spectrum between perfect correctness and total failure, enabling more nuanced evaluation and more informed model comparison.

## The Problem with Binary Metrics

### Exact Match: Too Strict

Exact Match compares the generated SQL string character-by-character against a reference query. If they are textually identical, the score is 1; otherwise, 0. The fundamental problem is that SQL is a declarative language with extensive syntactic flexibility. Consider these two queries:

```sql
-- Query A
SELECT name FROM users WHERE age > 25

-- Query B
SELECT u.name FROM users AS u WHERE u.age > 25
```

These queries are semantically identical — they return exactly the same results on any database. But Exact Match scores Query B as a failure if Query A is the reference. Table aliases, JOIN order, clause reordering, subquery versus JOIN reformulations — all of these produce functionally equivalent queries that EM treats as incorrect.

### Execution Accuracy: Too Coarse

Execution Accuracy improves on EM by actually running both queries and comparing result tables. If the outputs match, the score is 1; otherwise, 0. This handles syntactic variation elegantly, but introduces a different problem: **no partial credit**.

A query that returns 99 out of 100 correct rows receives the same score (0) as one returning completely irrelevant data. A query selecting the right columns from the right tables but with a slightly wrong filter condition is treated identically to one querying entirely wrong tables. These distinctions are critical for understanding model capabilities and guiding improvements, yet binary execution accuracy erases them entirely.

### The Practical Impact

For researchers, binary metrics make it impossible to distinguish between models that are "almost there" and models that are fundamentally off-track. Two models with 70% execution accuracy might have very different error profiles — one consistently making small mistakes, the other alternating between perfect outputs and complete failures. Binary metrics cannot distinguish these cases.

For practitioners evaluating deployment readiness, the difference between "usually close to correct" and "either perfect or useless" is enormous — but invisible under current metrics.

## The Query Accuracy Score (QAS)

QAS provides a continuous value between 0 and 1 by combining two complementary similarity measures:

### Semantic Similarity (S_C)

We measure the structural similarity between generated and reference queries using **code-specialized embedding models**. Specifically, we employ **UAE-Code-Large-V1**, a model trained to produce meaningful vector representations of code, including SQL.

The key design choice here is using code-specific rather than general-purpose text embeddings. Standard language models do not fully grasp SQL-specific constructs: the functional equivalence of different JOIN syntaxes, the semantic role of WHERE clauses versus HAVING clauses, or the meaning of subquery nesting. Code-specialized embeddings capture these nuances, producing similarity scores that correlate with actual functional similarity.

The cosine similarity between the embedding vectors of the generated and reference queries gives us S_C — a measure of how similar the two queries are in terms of their structural intent.

### Table Similarity (S_T)

While semantic similarity captures intent, table similarity captures outcomes. We execute both queries on the database and compare the resulting tables using an **edit-distance-based algorithm**.

This goes far beyond binary comparison. The algorithm computes the minimum number of edit operations (insertions, deletions, substitutions) needed to transform one result table into the other, normalized by the table size. A table missing one row gets a high similarity score; a table with completely different data gets a low score. The continuous nature of this measure provides the granularity that binary EX lacks.

We also demonstrated that simple structural proxies — such as comparing the number of rows or columns — are unreliable. Our analysis showed essentially **no correlation between differences in table dimensions and actual content similarity**. Tables with identical shapes can contain entirely different data, confirming the need for content-level comparison.

### Combining the Components

The final QAS is a weighted combination:

> QAS = w × S_T + (1 − w) × S_C

We analyzed the sensitivity of the weight parameter w using **Kendall distance** between model rankings at different settings. Rankings were stable across intermediate values (w = 0.25, 0.5, 0.75), indicating that QAS is robust to the specific weight choice. We selected **w = 0.5** to equally weight both components, providing a balanced measure of intent and outcome similarity.

## Experimental Evaluation

We evaluated QAS on the **BIRD benchmark**, a challenging dataset of real-world database queries across diverse domains. We assessed **11 text-to-SQL models**, including:

- Fine-tuned specialist models designed specifically for text-to-SQL
- General-purpose LLMs (GPT-4 and variants)
- Various open-source alternatives of different sizes

### Key Findings

**Hidden distinctions revealed.** Models that appeared equivalent under binary metrics showed meaningful differences under QAS. Two models with similar EX scores of around 65% turned out to have strikingly different error profiles: one produced consistently mediocre queries (moderate QAS scores across the board), while the other was more "all-or-nothing" (high QAS on successes, very low QAS on failures).

**Diagnostic capability.** The two-component structure of QAS enables differential diagnosis:
- **High S_C, low S_T**: The model understands the query intent but makes execution-level errors (wrong filter values, missing conditions). This suggests the model grasps SQL structure but struggles with precise value mapping.
- **Low S_C, high S_T**: Structurally different queries that happen to produce similar results. This can occur with equivalent reformulations or coincidental output matches.
- **Low S_C, low S_T**: Fundamental misunderstanding of the query requirements.

This diagnostic information is invaluable for targeted model improvement — a capability that binary metrics simply cannot provide.

**Stable rankings.** The model rankings produced by QAS were consistent across different weight configurations, suggesting the metric captures robust underlying quality differences rather than being an artifact of parameter choices.

## Broader Implications

For the research community, QAS enables more informative benchmarking. Instead of reporting a single binary accuracy number, researchers can characterize the full distribution of query quality, enabling more nuanced model comparisons and more targeted architectural improvements.

For practitioners, QAS provides a more honest assessment of model capabilities. A system with 70% binary accuracy and high average QAS on failures is fundamentally different from one with 70% accuracy and low average QAS on failures — and deployment decisions should reflect this difference.

Looking ahead, QAS could potentially serve as a **training objective**, providing continuous, differentiable feedback during model training rather than binary pass/fail signals. This could fundamentally change how text-to-SQL models learn, enabling gradient-based optimization toward query quality rather than relying solely on binary supervision.

---

*Published in Scientific Reports, 15.1: 22357, 2025. This research was conducted at the University of Trieste and NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa. Code available at [github.com/giovannipinna96/sql_metric](https://github.com/giovannipinna96/sql_metric).*
