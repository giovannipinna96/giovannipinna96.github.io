---
title: "GA4GC: Greener Agent for Greener Code"
date: 2025-10-13
draft: false
tags: ["Green AI", "AI Coding Agents", "Multi-Objective Optimization", "NSGA-II", "Sustainability"]
categories: ["Research"]
description: "Using multi-objective optimization to configure AI coding agents for energy-efficient operation, achieving up to 37.7% runtime reduction while improving code correctness through systematic configuration tuning with NSGA-II."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
AI coding agents consume substantial computational resources — often over 100,000 tokens per run — yet ship with default configurations that are far from optimal. GA4GC (Greener Agent for Greener Code) applies NSGA-II multi-objective optimization to systematically tune agent configurations, balancing code correctness, performance improvement, and agent runtime. Evaluated on a mini-SWE-agent architecture powered by Gemini 2.5 Pro against the SWE-Perf benchmark, the framework achieves up to 37.7% runtime reduction and a 135-fold hypervolume improvement over default settings, while simultaneously improving code correctness. Random Forest analysis reveals that temperature is the single most influential parameter, and that LLM hyperparameters primarily drive task effectiveness while agent constraints primarily drive resource consumption — a separation that enables independent tuning of quality and efficiency. Published at SSBSE 2025, Challenge Track on Green SBSE.
{{< /summary-box >}}

## Introduction

AI coding agents — tools like GitHub Copilot, Claude Code, Devin, and OpenAI Codex — represent a significant evolution beyond simple code completion. These systems operate through complex, multi-step reasoning pipelines: they analyze repository structure, plan solutions, generate code, execute it in sandboxed environments, diagnose failures, and iterate through multiple cycles of refinement. They can tackle real-world software engineering tasks that no single-shot LLM call could handle.

But this power comes at a substantial cost. A single agent run on a moderately complex software engineering task can consume over **100,000 tokens**, translating to significant monetary expense and energy consumption. And here lies a critical paradox: when an AI agent is tasked with *optimizing code performance*, the energy consumed by the agent itself during the optimization process can vastly exceed the energy saved by the resulting code improvements. Without careful configuration tuning, an agent might need to produce code that runs **hundreds of thousands of times** before the energy savings offset the optimization cost. Some "optimizations" are actually a net energy loss.

This paper, presented at **SSBSE 2025** (the 17th Symposium on Search-Based Software Engineering) as part of the Challenge Track on Green SBSE, introduces **GA4GC** (Greener Agent for Greener Code) — a framework that applies multi-objective optimization to find agent configurations that balance effectiveness with resource efficiency.

## The Configuration Space Problem

An AI coding agent has a surprisingly large configuration space. Key parameters include:

- **LLM temperature**: Controls the randomness of text generation (0 = deterministic, higher = more creative/random)
- **Top_p sampling**: Another diversity control parameter that limits token selection to the most probable options
- **Maximum token limits**: How many tokens the agent can generate per step
- **Step limits**: How many reasoning-action cycles the agent can perform
- **Prompt template variants**: Different instructions and system prompts that shape agent behavior

Each parameter affects both the quality of the agent's output and its resource consumption. Interactions between parameters are complex and often counterintuitive — a higher temperature might improve code quality on creative tasks but waste resources on straightforward ones. The combined configuration space is too large for manual exploration to be effective.

## How GA4GC Works

### The Agent Architecture

GA4GC operates on a **mini-SWE-agent** architecture powered by **Gemini 2.5 Pro** as the backbone LLM. The agent follows the standard SWE-agent workflow: reading repository files, planning modifications, generating patches, running tests, and iterating on failures. This architecture represents a realistic deployment scenario for AI coding agents in practice.

### The Optimization Framework

GA4GC frames configuration tuning as a **multi-objective optimization problem** and solves it using **NSGA-II** (Non-dominated Sorting Genetic Algorithm II). The three competing objectives are:

1. **Minimize incorrect patches**: Maximize the likelihood that the agent produces correct, functional code
2. **Maximize code performance improvement**: Ensure the optimized code actually runs faster — this is the primary purpose of the optimization task
3. **Minimize agent execution runtime**: Reduce the computational resources (time, tokens, energy) consumed by the agent itself

The search space is heterogeneous: continuous parameters (temperature, top_p), integer constraints (maximum tokens, step limits), and categorical variables (prompt templates). NSGA-II is well-suited for this kind of mixed-variable problem, applying appropriate crossover and mutation operators for each parameter type.

### Evaluation on SWE-Perf

Each candidate configuration is evaluated on tasks from the **SWE-Perf benchmark**, which provides authentic, repository-level performance optimization tasks from the **astropy** project (a widely-used Python library for astronomy). The agent generates patches that are validated in **isolated Docker environments**, ensuring reproducible measurements of both code correctness and performance gains.

### Evolutionary Process

Starting from a population of random configurations, NSGA-II evolves better solutions over multiple generations. Through selection, crossover, and mutation, the algorithm converges toward a **Pareto front** of non-dominated configurations — solutions where improving one objective necessarily worsens another. This Pareto front gives practitioners a menu of trade-off options to choose from.

## Key Results

Under a constrained budget of just **25 configuration evaluations**, GA4GC achieved remarkable results:

### Efficiency Gains

Non-dominated configurations achieved up to **37.7% runtime reduction** (943 seconds vs. 1,513 seconds for the default configuration) while simultaneously *improving* code correctness. The **hypervolume improvement** — a measure of how well the Pareto front covers the objective space — reached up to **135-fold** compared to the default baseline.

This is a critical finding: it means the default configurations shipped with AI coding agents are far from optimal. Significant improvements in both quality and efficiency are available through systematic tuning.

### Parameter Importance Analysis

Using **Random Forest regression analysis**, we identified which parameters matter most:

**Temperature is the most influential parameter overall.** This makes intuitive sense: temperature controls the fundamental randomness of LLM outputs, affecting everything from code creativity to exploration breadth. Small changes in temperature can dramatically alter agent behavior.

The analysis revealed an important structural finding — **two categories of parameters serve different roles**:

- **LLM hyperparameters** (temperature, top_p) primarily impact **task effectiveness** — whether the agent produces correct and performant code
- **Agent constraints** (token limits, step counts) primarily impact **resource consumption** — how much time and compute the agent uses

This separation is highly actionable: practitioners can tune resource usage without necessarily affecting code quality, and vice versa. It means efficiency improvements and quality improvements can often be pursued independently.

## Practical Deployment Strategies

GA4GC translates the optimization results into three concrete deployment scenarios:

### 1. Runtime-Critical Environments

Use **low temperature** with **restrictive top_p**. The agent generates less diverse but faster code, completing tasks with minimal computational overhead. Ideal when turnaround time matters most and tasks are relatively straightforward.

### 2. Performance-Critical Scenarios

Use **moderate temperature** (0.65–0.73) with **balanced top_p**. This gives the agent enough exploratory freedom to discover genuinely better solutions, at the cost of higher resource consumption. Appropriate when the code performance gains justify the additional agent runtime.

### 3. Context-Specific Optimization

Run GA4GC itself on your specific codebase and task distribution. The Pareto front it discovers will be tailored to your particular needs, providing the best possible trade-offs for your environment. This represents the most thorough approach for organizations where agent usage is frequent and optimization matters.

## Why This Matters

As AI coding agents transition from experimental tools to standard development infrastructure, their cumulative computational footprint becomes a sustainability concern. An organization running hundreds of agent tasks per day generates significant energy costs — both financially and environmentally.

GA4GC demonstrates that **configuration tuning is a sustainability lever** that complements traditional approaches like model architecture improvements or hardware optimization. Rather than accepting default configurations and absorbing the cost, practitioners can systematically discover configurations that deliver the performance they need while minimizing waste.

The framework provides a methodological template for responsible AI deployment in software engineering: measure, optimize, and deploy with awareness of the full cost-benefit picture.

---

*Published at the 17th Symposium on Search-Based Software Engineering (SSBSE 2025), Challenge Track on Green SBSE. This research was conducted at University College London (UCL) and the University of Trieste. Code and results are publicly available.*
