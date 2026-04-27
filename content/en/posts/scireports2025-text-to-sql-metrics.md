---
title: "The Text-to-SQL Field Has a Measurement Problem"
date: 2025-07-02
draft: false
tags: ["Text-to-SQL", "Evaluation Metrics", "Semantic Similarity", "SQL", "LLM"]
categories: ["Research"]
description: "Every text-to-SQL benchmark today scores queries as either perfect or wrong. That's a coin flip dressed up as a metric. We built one that actually tells you how close you got."
ShowToc: true
TocOpen: false
cover:
  image: "/images/scireports2025-text-to-sql-metrics/Figure2.png"
  alt: "QAS — Query Accuracy Score combining semantic and table similarity"
  hiddenInList: false
---

{{< summary-box title="TL;DR" >}}
Text-to-SQL is everywhere, but we measure it badly. **Exact Match** punishes you for swapping `users AS u`. **Execution Accuracy** doesn't care if you got 99 of 100 rows right — wrong is wrong. We built **QAS (Query Accuracy Score)**: a continuous score that combines code-aware semantic similarity (how close is the SQL?) with edit-distance table similarity (how close is the answer?). Tested on 11 models on BIRD, QAS surfaces *huge* differences that binary metrics flatten into the same number.
{{< /summary-box >}}

## A field built on coin flips

Text-to-SQL is one of those areas where the demos look magical. Type a question in English, get a SQL query back, get an answer from your database. No DBA needed. The promise is enormous.

The progress is real. The *measurement* of the progress is not.

Look at any text-to-SQL leaderboard and you'll see two metrics doing all the work: **Exact Match** and **Execution Accuracy**. Both are binary. A query is correct or it's not. There is no in-between.

This is a problem. Most queries that "fail" are not actually random nonsense — they're 80% right, with a wrong filter, or a missing column, or an extra row. Treating them as identical to "completely wrong query against the wrong table" throws away exactly the information we need to make models better.

## Why Exact Match is a bad joke

Exact Match compares two SQL strings character by character. Same string, score 1. Different string, score 0.

```sql
-- Reference
SELECT name FROM users WHERE age > 25

-- Generated
SELECT u.name FROM users AS u WHERE u.age > 25
```

These return identical results on every row of every database in the universe. Exact Match scores the second one as a complete failure.

SQL is a *declarative* language. There are dozens of valid ways to write the same query. Aliases, JOIN order, subquery vs. JOIN, WHERE vs. HAVING — all syntactic flexibility, all semantically equivalent, all torpedoed by a metric that compares strings.

## Why Execution Accuracy is better but still wrong

Execution Accuracy at least runs both queries and compares the result tables. If the rows match, score 1. Otherwise, 0.

This handles the alias problem elegantly. It also recreates a different one: **no partial credit**.

A query that returns 99 of 100 correct rows scores zero. A query that selects the right columns from the right tables but with one slightly off filter scores zero. A query against completely the wrong tables — also zero.

These are not the same kind of failure. Treating them as one is destroying signal that researchers and practitioners desperately need.

## QAS: a continuous score with two eyes

We built **QAS — the Query Accuracy Score** — to fix this. It's a number between 0 and 1, and it has two components measuring different things:

![QAS pipeline: semantic similarity meets table similarity](/images/scireports2025-text-to-sql-metrics/Figure2.png)

**S_C — Semantic Similarity.** How close are the *queries themselves*? We embed both queries with **UAE-Code-Large-V1**, a model trained specifically on code. General-purpose text embeddings don't understand SQL — they don't know that LEFT JOIN and RIGHT JOIN aren't synonyms, or that subqueries can be functionally identical to JOINs. Code-specialized embeddings do. We take the cosine similarity of the embeddings. That's S_C.

**S_T — Table Similarity.** How close are the *results*? We run both queries and compare the result tables with edit distance — the minimum number of insertions, deletions, and substitutions to transform one table into the other, normalized by size. Off by one row? High S_T. Off by every value? Low S_T.

The final score is just:

> QAS = w · S_T + (1 − w) · S_C

We tested how sensitive the ranking is to *w* using Kendall distance and the answer was: not very. Rankings are stable for w ∈ [0.25, 0.75]. We picked **w = 0.5** because we wanted intent and outcome to count equally.

A small but important sub-result: simple proxies like "do the result tables have the same number of rows?" don't work. We measured the correlation between table-shape similarity and actual content similarity and it was essentially zero. **Two tables with identical shape can contain entirely different data.** Edit distance over the actual content is necessary.

## What QAS shows that binary metrics hide

We ran QAS against 11 text-to-SQL models on the BIRD benchmark — fine-tuned specialists, general-purpose GPT-4-class systems, open-source models of various sizes.

Two findings stood out.

**Hidden differences in "equivalent" models.** Two models with the same Execution Accuracy of ~65% can have completely different *shapes* of failure. One was a reliable mediocre — close every time, but rarely perfect. The other was bipolar — perfect or catastrophic, nothing in between. Binary metrics call these "the same model." They are not.

**Diagnostic power.** The two components of QAS act like a tiny diagnostic kit:

- **High S_C, low S_T** → the model understood the query but messed up the values (wrong filter, missing condition). It speaks SQL, but it can't do the math.
- **Low S_C, high S_T** → structurally different query, similar output. Either a clever reformulation or an accidental match.
- **Low S_C, low S_T** → the model didn't understand the question.

That distinction matters. The first case is fixable with better grounding on database content. The third needs better intent understanding. Binary metrics give you neither — just "wrong."

## What this enables

For researchers: stop reporting one number. Report a *distribution*. Showing that your model has higher mean QAS on failures than the baseline is a meaningful claim, even when binary accuracy is identical.

For practitioners: a 70% accurate model with high mean QAS on its failures is *very different* from a 70% accurate model with low mean QAS on its failures. The first is "close most of the time, ship it carefully." The second is "binary success, plan for fallback." Deployment decisions hinge on this.

For training: QAS is continuous and differentiable in spirit. It could be used as a richer reward signal than the pass/fail signals models train against today. We don't have to settle for binary supervision when our evaluation metric finally has texture.

The TL;DR is simple. **You can't optimize what you can't see.** Binary metrics can't see the gradient of "almost right." QAS can.

---

### Reference

This post is a divulgative summary of:

> Pinna, G., Manzoni, L., De Lorenzo, A., Castelli, M. (2025). *Beyond Exact Set and Execution Matches: Redefining Text-to-SQL Metrics with Semantic and Structural Similarity*. **Scientific Reports**, 15(1): 22357.
>
> [Read the original paper (PDF)](/images/scireports2025-text-to-sql-metrics/2025_Scientific_Reports_Beyond_Exact_Set_and_Execution_Matches__Redefining_Text_to_SQL_Metrics_with_Semantic_and_Structural_Similarity.pdf) — [Code on GitHub](https://github.com/giovannipinna96/sql_metric)

*Research conducted at the University of Trieste and NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa.*
