---
title: "Analyzing Message-Code Inconsistency in AI Coding Agent-Authored Pull Requests"
date: 2026-04-14
draft: false
tags: ["AI Coding Agents", "Pull Requests", "Message-Code Inconsistency", "Trust", "MSR"]
categories: ["Research"]
description: "A large-scale analysis of 23,247 AI-authored pull requests revealing that message-code inconsistency — when PR descriptions don't match actual code changes — leads to 51.7% lower acceptance rates and 3.5× longer merge times."
ShowToc: true
TocOpen: false
---

{{< summary-box title="Abstract" >}}
AI coding agents generate both code and pull request descriptions, but these two outputs can diverge: the code may be correct while the description inaccurately reflects the actual changes. We study this message-code inconsistency (MCI) phenomenon across 23,247 AI-authored pull requests, finding that 1.7% exhibit high inconsistency between their description and their code diff. Despite the low prevalence, the impact is dramatic: inconsistent PRs experience 51.7% lower acceptance rates and 3.5 times longer merge times compared to consistent ones — even when the underlying code changes are technically sound. This occurs because misleading descriptions erode reviewer trust and force costly re-orientation during the review process. Our findings highlight that evaluating AI agents on code quality alone provides an incomplete picture; the accuracy of their communication artifacts is equally critical for practical utility. Published at MSR 2026, Mining Challenge.
{{< /summary-box >}}

## Introduction

When we evaluate AI coding agents, our attention naturally gravitates toward code quality. Does the generated code compile? Does it pass tests? Is it well-structured and maintainable? These are important questions, but they capture only part of what makes a pull request successful.

In professional software development, a pull request is not just a code diff — it is a **communication artifact**. It includes a description explaining what changes were made, why they were made, and what impact they are expected to have. Reviewers rely heavily on these descriptions as their first point of entry into understanding a contribution. Before reading a single line of code, most reviewers read the PR description to form a mental model of what to expect.

This creates a critical dependency: **if the description accurately reflects the code, it accelerates review; if it doesn't, it actively misleads.** A PR description claiming "fixed the authentication bug in the login flow" that actually contains a refactoring of the database connection layer sends the reviewer down the wrong path from the start.

This is the problem of **message-code inconsistency (MCI)** — the misalignment between the natural language description of a pull request and the actual code changes it contains. This paper, published at **MSR 2026** (the 23rd International Conference on Mining Software Repositories), presents the first large-scale study of this phenomenon in AI-authored pull requests.

## Why AI Agents Are Particularly Prone to MCI

AI coding agents typically generate both code and PR descriptions using the same or similar LLM backbone. But writing correct code and writing accurate descriptions are fundamentally different cognitive tasks.

Writing code requires **algorithmic reasoning**: understanding the problem specification, choosing an appropriate approach, implementing it with correct syntax and semantics, and handling edge cases. Writing an accurate description requires **meta-cognitive awareness**: understanding the *relationship* between what was intended, what was attempted, and what was ultimately achieved.

The inconsistency often arises from a specific failure mode in agent behavior. During execution, an agent's plan may diverge from its actual implementation. The agent might:

1. Start with a clear plan based on the task description
2. Encounter unexpected difficulties during implementation (failing tests, compilation errors, dependency issues)
3. Iterate through multiple debugging and code modification cycles
4. Arrive at a final solution that differs from the original plan

When the agent then generates a PR description, it may describe **what it intended to do** (based on the initial plan) rather than **what it actually did** (the outcome of the complex, sometimes meandering implementation process). The description reflects the plan; the code reflects the outcome — and these can diverge significantly.

## Study Design

### Scale and Scope

We analyzed **23,247 pull requests** authored by AI coding agents, measuring the degree of inconsistency between each PR's description and its actual code changes. This is, to our knowledge, the largest study of message-code consistency in AI-generated contributions.

### The PR-MCI Metric

We developed a metric called **PR-MCI (Pull Request Message-Code Inconsistency)** to quantify the alignment gap. PR-MCI measures the semantic distance between what a PR description claims and what the code diff actually does, producing a continuous score that captures the degree of misalignment.

### Prevalence of Inconsistency

Our analysis found that **1.7% of AI-authored pull requests exhibited high message-code inconsistency**. While this number might appear small in isolation, two factors make it significant:

1. **At scale, 1.7% represents a substantial absolute number.** In a large organization generating thousands of AI-authored PRs per month, this translates to dozens of misleading descriptions entering the review pipeline regularly.

2. **The impact of each inconsistent PR is disproportionately large**, as our outcome analysis demonstrates.

## The Impact of Inconsistency

Pull requests with high MCI scores showed dramatically worse outcomes across two key dimensions:

### Acceptance Rates

PRs with high message-code inconsistency had **51.7% lower acceptance rates** compared to PRs with consistent descriptions. This is a striking finding: even when the code changes themselves might be perfectly acceptable, a misleading description causes reviewers to reject the contribution.

This happens for several reasons. When reviewers detect that a description doesn't match the code, they lose trust in the entire contribution. If the agent can't accurately describe its own changes, how confident can the reviewer be that the code is correct? The description inconsistency serves as a **negative signal about overall quality**, even when the code itself is fine.

Additionally, inconsistent descriptions make it much harder for reviewers to evaluate the code. A reviewer who expects to see authentication fixes but finds database refactoring must re-orient their mental model entirely — a cognitively expensive and frustrating experience that naturally leads to higher rejection rates.

### Merge Time

PRs with high MCI scores took **3.5 times longer to merge** compared to consistent PRs. This bottleneck arises because inconsistent descriptions force reviewers to do fundamentally more work:

- Instead of being guided by an accurate summary, reviewers must read through the entire code diff line by line
- They need to construct their own understanding of what the changes do, rather than verifying a provided explanation
- Additional review rounds may be needed to clarify discrepancies between the description and the code

In aggregate, these delays create significant friction in the development pipeline. When AI agents are expected to accelerate development, producing PRs that slow down the review process directly undermines their value proposition.

## Implications

### For AI Agent Developers

The findings make a strong case for investing in **description verification mechanisms**. Agent developers should implement a separate validation step that checks whether the generated PR description accurately reflects the actual code changes. This could take several forms:

- **Secondary LLM pass**: A separate model (or the same model in a distinct context) reads both the code diff and the generated description, flagging inconsistencies
- **Heuristic checks**: Lightweight rules that verify basic alignment (e.g., files mentioned in the description actually appear in the diff, described bug types match the actual error patterns)
- **Post-generation description regeneration**: Instead of using the description generated during the planning phase, regenerate the description from the final code diff to ensure it reflects the actual outcome

### For Development Teams

Teams using AI coding agents should calibrate their review processes to account for potential description unreliability. In high-stakes codebases, this might mean:

- Developing a systematic habit of verifying PR descriptions against the actual code changes before beginning detailed review
- Implementing automated tools that flag potential description-code mismatches
- Considering independent description generation through a separate process

### For Researchers

This study highlights that evaluating AI coding agents on code quality alone provides an incomplete picture. The full deliverable package includes code, commit messages, PR descriptions, and documentation. Inconsistency in any of these components can undermine the practical utility of the agent's work, even when the code itself is technically sound. Future evaluation frameworks should assess these artifacts holistically.

## The Trust Dimension

As AI coding agents move from assistants to increasingly autonomous contributors, **trust** becomes a central concern. Trust in software development is not built solely on code correctness — it requires transparency and honest communication about what changes are being made and why.

Our findings reveal that current agents have meaningful room for improvement on this dimension. The code may be technically correct, but the narrative they construct about their own work is not always reliable. Addressing message-code inconsistency is an important step toward AI agents that development teams can trust — not just to write good code, but to communicate clearly and honestly about what they have done.

---

*Published at the 23rd International Conference on Mining Software Repositories (MSR 2026) — Mining Challenge. This research was conducted at University College London (UCL) and King's College London.*
