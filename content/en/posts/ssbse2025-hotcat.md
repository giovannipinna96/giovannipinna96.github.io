---
title: "HotCat: Green and Effective Feature Selection for Hotfix Bug Taxonomy"
date: 2025-10-13
draft: false
tags: ["Bug Taxonomy", "NSGA-II", "Multi-Objective Optimization", "Green AI", "Feature Selection"]
categories: ["Research"]
description: "A multi-objective optimization approach for classifying software hotfixes that balances classification quality with computational efficiency, demonstrating that Green AI principles can be applied without sacrificing effectiveness."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
Classifying software hotfixes into bug categories is challenging due to sparse data, severe class imbalance, and the high computational cost of LLM-based code analysis. HotCat addresses these challenges through NSGA-II multi-objective optimization, treating feature selection as a search problem over 18 available features extracted from the HotBugs dataset (88 hotfix entries across 17 categories). The framework simultaneously optimizes classification accuracy, Normalized Mutual Information, and computational runtime. A two-stage data augmentation strategy improves generalization from 55% to 72%. The resulting Pareto front reveals that classification quality and efficiency need not be in conflict: a balanced configuration achieves 59% accuracy and 0.58 NMI in just 129 seconds, while selective feature pruning actually improves results by removing noise-introducing features. Published at SSBSE 2025, Challenge Track on Hot Fixing Benchmark.
{{< /summary-box >}}

## Introduction

In software engineering, not all bugs are created equal. While some defects can be queued for the next scheduled release, others demand immediate attention. These urgent patches — known as **hotfixes** — address critical issues requiring rapid deployment to production: security vulnerabilities, payment processing failures, service outages, or data corruption bugs affecting live users.

Understanding the nature and distribution of these hotfixes is essential for software teams. A well-constructed **bug taxonomy** — a systematic classification of bug types — helps teams prioritize resources, identify recurring failure patterns, and implement preventive measures. But building such taxonomies is challenging, particularly for hotfixes: the data is sparse (hotfixes are a small fraction of all patches), the class distribution is severely imbalanced (some bug types are far rarer than others), and accurate classification requires sophisticated semantic analysis of code changes.

This paper, presented at **SSBSE 2025** (the 17th Symposium on Search-Based Software Engineering) as part of the Challenge Track on Hot Fixing Benchmark, introduces **HotCat** — a framework that addresses these challenges while adhering to **Green AI** principles by minimizing unnecessary computational expense.

## The Green AI Motivation

Modern approaches to code analysis increasingly rely on Large Language Models for semantic understanding — summarizing code changes, generating embeddings, and classifying intent. These models are powerful but computationally expensive: every LLM inference consumes energy, and when processing thousands of patches across dozens of features, the cumulative cost becomes significant.

HotCat asks a pointed question: **do we need all available features to achieve good classification, or can we selectively prune the feature space to reduce computational cost without sacrificing quality?** This is not merely an efficiency concern — it is an environmental and economic imperative as LLM-based analysis tools become standard in development workflows.

## The HotCat Pipeline

### Data Foundation: The HotBugs Dataset

HotCat operates on the **HotBugs** dataset, which contains **88 hotfix entries** spanning **17 bug categories** extracted from real-world software projects. Each entry is a code patch associated with metadata from Jira issue tracking, providing both the raw code changes and contextual information about each fix.

### Feature Engineering

Starting from the raw data, HotCat enriches the feature space by integrating:

- **Code-level features**: Extracted from the actual diffs — lines added, lines removed, files changed, syntactic complexity
- **Project metadata from Jira**: Time-to-fix, number of participants (developers, reviewers), priority levels, and other organizational signals
- **LLM-generated summaries**: Each hotfix is summarized using an LLM to produce concise natural language descriptions of what the patch does

These summaries are then transformed into dense vector representations using **Sentence-BERT embeddings**, which capture the semantic content of each description. The vectors are organized through **K-Means clustering** to produce the actual classification.

In total, **18 features** are available for the classification pipeline, creating a search space of 2^18 (over 260,000) possible feature combinations.

### Multi-Objective Feature Selection with NSGA-II

This is the core innovation. Rather than using all 18 features or manually selecting a subset, HotCat formulates feature selection as a **multi-objective optimization problem** and solves it with **NSGA-II** (Non-dominated Sorting Genetic Algorithm II), implemented using the **pymoo** library.

Each candidate solution is represented as a **binary bitmask** — a vector of 0s and 1s indicating which features to include. NSGA-II simultaneously optimizes three objectives:

1. **Maximize classification accuracy**: How well the selected features enable correct bug categorization
2. **Maximize Normalized Mutual Information (NMI)**: A measure of agreement between the predicted clustering and ground truth labels, robust to cluster size imbalance
3. **Minimize computational runtime**: How quickly the classification pipeline executes with the selected features

The evolutionary search uses a **population of 20 individuals** evolved over **20 generations**, with binary crossover and bit-flip mutation operators suited to the binary encoding.

Through this process, NSGA-II discovers a **Pareto front** of non-dominated solutions — configurations where improving one objective necessarily worsens another. This gives practitioners a menu of options to choose from based on their specific constraints.

### Data Augmentation for Robustness

The HotBugs dataset's small size (88 entries) and severe class imbalance pose challenges for any classification approach. HotCat addresses this with a **two-stage augmentation strategy**:

1. **Category balancing**: Synthetic examples are generated to equalize the representation of rare bug categories
2. **Post-optimization record generation**: Additional data is created after the feature selection stage to improve generalization

This augmentation proved crucial: **generalization performance improved from 55% to 72%** — a 17 percentage-point gain that demonstrates the importance of addressing data scarcity in this domain.

## Results

The Pareto front revealed several practically useful operating points:

- **Balanced configuration**: **59% accuracy** and **0.58 NMI** with a runtime of just **129 seconds** — offering good classification quality at minimal computational cost
- **Maximum accuracy configuration**: **63% accuracy** in **132 seconds** — only 3 additional seconds for a 4 percentage-point improvement in accuracy

These results demonstrate a key finding: **classification quality and computational efficiency do not have to be in conflict**. Higher accuracy was achievable without dramatic increases in resource consumption.

The analysis also revealed which features matter most. Not all metadata fields contribute equally to classification quality — some features actually **degrade performance** by introducing noise. By selectively pruning these features, HotCat achieves better results with less computation, embodying the Green AI principle of doing more with less.

## Implications

HotCat demonstrates a practical methodology for **Green AI in software engineering**. As LLM-based analysis tools become embedded in development workflows, their cumulative energy footprint becomes a genuine concern. Multi-objective feature selection offers a principled approach to keeping this footprint in check.

The framework is designed to be replicable and scalable, offering teams a method for automating hotfix analysis within issue tracking systems like Jira while respecting computational constraints. Future work will expand the approach to larger datasets and explore incorporating direct carbon emission metrics as optimization objectives.

---

*Published at the 17th Symposium on Search-Based Software Engineering (SSBSE 2025), Challenge Track on Hot Fixing Benchmark. This research was conducted at University College London (UCL) and the University of Trieste.*
