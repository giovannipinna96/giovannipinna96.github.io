---
title: "There Is No \"Best\" AI Coding Agent — And That's the Whole Point"
date: 2026-04-14
draft: false
tags: ["AI Coding Agents", "Pull Requests", "Empirical Study", "Software Engineering", "MSR"]
categories: ["Research"]
description: "We looked at 7,156 pull requests from five AI coding agents on real open-source projects. The agent matters less than you'd think. The kind of work matters far more."
ShowToc: true
TocOpen: false
cover:
  image: "/images/post-placeholder.svg"
  alt: "Acceptance rates across AI coding agents and task types"
  hiddenInList: false
---

{{< summary-box title="TL;DR" >}}
We pulled **7,156 real pull requests** authored by Codex, Claude Code, Cursor, Devin, and Copilot, then sliced the data by *task type*. The headline: the gap between best and worst task category is **29 percentage points** — far bigger than the gap between agents inside any single category. Codex is the steady generalist. Claude Code wins documentation. Cursor wins bug fixes. Stop asking *which agent is best*. Start asking *best at what*.
{{< /summary-box >}}

## The wrong question

Walk into any engineering Slack and you'll see the same debate: Codex vs. Claude Code vs. Cursor vs. Devin vs. Copilot. Threads explode. Benchmarks fly. Someone screenshots a leaderboard.

The honest answer to "which is best" is *it depends* — but that answer feels like a cop-out. So we tried to make it concrete. We grabbed 7,156 pull requests these five agents had actually opened against real open-source repos, and we measured what really matters: **did a human maintainer merge it?**

Then we did the part most evaluations skip. We split the PRs by what kind of work they were doing.

## Why a single number lies

Bug fixes, feature work, documentation, refactors, dependency bumps, tests — these are not the same task. They have wildly different difficulty profiles for an AI agent. Documentation is mostly natural-language generation against a soft target. Feature implementation requires holding a mental model of an architecture. Bug fixing demands navigating someone else's code.

If you average across all of them, you get a number. You also lose the entire story.

So instead of one acceptance rate per agent, we computed an acceptance rate **per agent, per task type**.

![Acceptance rates by AI coding agent and task type](/images/msr2026-comparing-ai-agents/Figure1.png)

## The headline finding

Look at the chart. The thing your eye should jump to is *vertical*, not horizontal.

The gap between the best task category and the worst is roughly **29 percentage points**. The gap between agents *within* any one category is far smaller. **The kind of work you give an agent predicts acceptance much more strongly than which agent you picked.**

That reframes the entire conversation. Asking "is Cursor better than Claude Code?" is like asking "is a chef better than a sushi chef?" — the question is missing a noun. *Better at what?*

## Each agent has a beat

When you slice properly, the agents stop blurring together. They develop personalities.

![Per-agent strengths across task types](/images/msr2026-comparing-ai-agents/Figure2.png)

**Codex** — the generalist. No spectacular peaks, no embarrassing valleys. If you only get to pick one agent and you don't know what's coming next, this is probably the safe default.

**Claude Code** — documentation. Its language fluency translates straight into PR descriptions, README rewrites, and inline comments that maintainers actually want to merge.

**Cursor** — bug fixes. The IDE integration gives it deeper context about the surrounding code, and that context is exactly what bug fixing needs.

Devin and Copilot land elsewhere on this map — the full breakdown is in the paper. The point isn't the leaderboard, it's that **there's no single ranking**. There's a shape.

## What to actually do with this

If you ship code with AI agents, two things follow:

**Route by task, not by preference.** A team that hands every kind of work to its favorite agent is leaving acceptance rate on the table. Documentation PR? Send it to Claude Code. Subtle bug? Cursor. Random Tuesday cleanup? Codex. Treat your agents like a team with different specialties, because that's what they are.

**Calibrate your expectations.** When a feature-implementation PR fails review, that may not be the agent's fault — it may be the *task's* fault. A 40% acceptance rate on hard tasks isn't a broken agent. A 70% rate on easy tasks isn't a magical one. Both are noise around a baseline set by the work itself.

## What this changes for the field

For agent developers: *stop chasing aggregate benchmark scores*. They hide where you're losing. The path to a better agent runs through the task category where you currently underperform.

For researchers: report per-task numbers, always. A single acceptance rate is not a measurement, it's a smoothing function over your most interesting variation.

And for everyone else: when someone tells you their agent is the best, ask them *at what*. If they can't answer, neither can their agent.

---

### Reference

This post is a divulgative summary of:

> Pinna, G., Sarro, F. (2026). *Comparing AI Coding Agents: A Task-Stratified Analysis of Pull Request Acceptance*. In: **Proceedings of the 23rd International Conference on Mining Software Repositories (MSR 2026)** — Mining Challenge Track.
>
> [Read the original paper (PDF)](/images/msr2026-comparing-ai-agents/MSR'26%20Mining%20Challenge%20-%20Comparing%20AI%20Coding%20Agents_%20A%20Task-Stratified%20Analysis%20of%20Pull%20Request%20Acceptance.pdf)

*Research conducted at University College London (UCL), CREST, with Prof. Federica Sarro.*
