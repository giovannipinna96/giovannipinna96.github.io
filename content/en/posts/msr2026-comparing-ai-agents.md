---
title: "Comparing AI Coding Agents: A Task-Stratified Analysis of Pull Request Acceptance"
date: 2026-04-14
draft: false
tags: ["AI Coding Agents", "Pull Requests", "Empirical Study", "Software Engineering", "MSR"]
categories: ["Research"]
description: "A large-scale empirical study of 7,156 pull requests across five AI coding agents, revealing that task type is the dominant factor in PR acceptance — more than the choice of agent — and that no single agent wins across all task categories."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
We present a large-scale empirical study of 7,156 pull requests authored by five AI coding agents — OpenAI Codex, Claude Code, Cursor, Devin, and GitHub Copilot — in real open-source repositories. By stratifying results by task type (bug fixing, feature implementation, documentation, refactoring, dependency updates, and testing), we discover that task type is the dominant factor in PR acceptance, with a 29 percentage-point gap between the best and worst task categories — far exceeding the variance between agents within any single category. No single agent outperforms all others across all task types: Codex achieves the most consistent general-purpose results, Claude Code leads in documentation tasks, and Cursor excels at bug fixing. These findings suggest that development teams should adopt an "agent portfolio" approach, matching each agent to the tasks where it performs best. Published at MSR 2026, Mining Challenge.
{{< /summary-box >}}

## Introduction

AI coding agents have transitioned from research prototypes to production tools with remarkable speed. GitHub Copilot, OpenAI Codex, Devin, Cursor, Claude Code — these systems are no longer confined to writing code snippets or completing function bodies. They create entire pull requests, implement features end-to-end, fix bugs across multiple files, and update documentation with minimal human supervision.

This rapid adoption has produced an inevitable question: **how do these agents actually compare in real-world practice?** Most existing evaluations answer this through controlled benchmarks — synthetic problems designed to test specific capabilities under standardized conditions. While valuable, benchmarks have well-known limitations: they may not reflect the diversity, messiness, and complexity of real-world software engineering tasks.

This paper, published at **MSR 2026** (the 23rd International Conference on Mining Software Repositories), takes a different approach. Rather than benchmarking agents on synthetic tasks, we studied their performance on **real pull requests** in **real open-source repositories**, using acceptance rates as the ultimate signal of practical utility.

## Study Design

### Dataset

We analyzed **7,156 pull requests** authored by **five AI coding agents** across a diverse set of open-source repositories. Each pull request represents a complete unit of work — code changes, commit messages, and PR descriptions — submitted to a real project with real maintainers making real acceptance decisions.

### Task Stratification

The key methodological contribution of this study is **task stratification**. Rather than computing a single overall acceptance rate per agent (which obscures important variation), we categorized each pull request by the type of task it addresses:

- **Bug fixing**: Correcting defects in existing code
- **Feature implementation**: Adding new functionality
- **Documentation**: Updating or creating documentation
- **Refactoring**: Restructuring code without changing behavior
- **Dependency updates**: Upgrading libraries and dependencies
- **Testing**: Adding or improving test coverage

This stratification enables a much richer comparison: instead of asking "which agent is best overall?", we can ask "which agent is best for each type of work?"

## The Key Finding: Task Type Dominates

Our most striking result is that **task type is the dominant factor influencing whether an AI-generated pull request gets accepted**. The acceptance rate gap between the best-performing and worst-performing task categories reached **29 percentage points** — substantially exceeding the variance between different agents within any single category.

This finding has profound implications for how we think about AI agent evaluation. The common framing of "which agent is best?" turns out to be somewhat misleading. The more useful question is: "which agent is best for *this specific type of task*?"

Different task categories have inherently different difficulty levels for AI agents. Documentation tasks, which primarily involve generating and editing natural language with relatively well-defined structure, tend to have higher acceptance rates. Feature implementation tasks, which require understanding complex system architecture and making decisions about design trade-offs, tend to have lower acceptance rates. These differences in inherent difficulty dwarf the differences between agents.

## Agent-Specific Strengths

Our task-stratified analysis revealed that **no single agent outperforms all others across all task types**. Instead, each agent has genuine areas of strength:

### OpenAI Codex

Codex achieved consistently high acceptance rates across most task categories, making it a solid **general-purpose choice**. Its strength lies in its versatility — it does not have dramatic peaks or valleys across task types, performing reliably regardless of what type of work is being done.

### Claude Code

Claude Code showed particular strength in **documentation tasks**, where its sophisticated language generation capabilities translate directly into high-quality prose. Documentation updates, README improvements, and inline comment generation all benefited from Claude Code's natural language fluency.

### Cursor

Cursor excelled specifically in **bug-fixing scenarios**. Its IDE-integrated workflow, which provides deep context about the surrounding codebase during the fix process, appears to give it an advantage when the task requires navigating and understanding existing code to identify and correct defects.

### The Heterogeneous Landscape

These findings paint a picture of a genuinely heterogeneous landscape. Different agents have different cognitive profiles — much like human developers who specialize in different aspects of software engineering. A developer who excels at debugging may not be the best choice for writing documentation, and vice versa. The same applies to AI agents.

## Practical Implications

### For Development Teams

The practical takeaway is twofold:

1. **Match the agent to the task.** Rather than defaulting to a single agent for all work, development teams can achieve better results by routing different types of work to the agents best suited for them. Bug fixes go to one agent, documentation updates to another, feature implementations to a third. This "agent portfolio" approach can meaningfully improve overall acceptance rates.

2. **Set realistic expectations by task type.** Some categories of work are inherently harder for AI agents than others. Understanding these baselines helps teams calibrate their review processes and avoid frustration when agents struggle with certain task types. If feature implementation PRs have a 40% acceptance rate while documentation PRs have a 69% acceptance rate, the difference reflects task difficulty, not agent quality.

### For Agent Developers

The task-stratified analysis provides specific guidance for agent improvement. Rather than optimizing for aggregate benchmarks, agent developers can focus on the task categories where their agent underperforms relative to competitors. This targeted improvement approach is more efficient and more likely to produce meaningful capability gains.

### For Researchers

Our findings highlight the critical importance of **task-stratified evaluation** in AI agent research. Aggregate metrics — overall acceptance rate, average benchmark score — can mask important performance variations that only become visible when results are broken down by task type. Future evaluations of AI coding agents should routinely report task-level performance to give a complete and honest picture.

## The Bigger Picture

As AI coding agents become increasingly integrated into software development workflows, understanding their actual capabilities and limitations — not as measured by synthetic benchmarks, but as observed in real-world deployment — is essential. Our study contributes a large-scale empirical foundation for this understanding, demonstrating that the question "which agent should I use?" has a more nuanced answer than most practitioners realize.

---

*Published at the 23rd International Conference on Mining Software Repositories (MSR 2026) — Mining Challenge. This research was conducted at University College London (UCL).*
