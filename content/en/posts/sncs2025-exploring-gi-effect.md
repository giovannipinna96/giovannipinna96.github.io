---
title: "Making the LLM-Plus-Evolution Pipeline Actually Smart"
date: 2025-07-01
draft: false
tags: ["Genetic Improvement", "LLM", "Lexicase Selection", "Down-Sampling", "Code Generation"]
categories: ["Research"]
description: "Last year we showed evolution can fix LLM code. This year we made the evolution itself smarter — better selection, partial credit, fewer cycles — and got improvements in 11 of 12 cases."
ShowToc: true
TocOpen: false
cover:
  image: "/images/post-placeholder.svg"
  alt: "Enhanced GI pipeline with lexicase selection and refined fitness"
  hiddenInList: false
---

{{< summary-box title="TL;DR" >}}
Our EuroGP 2024 work showed Genetic Improvement (GI) can rescue LLM-generated code. This follow-up makes the GI part itself smarter. Three upgrades: **lexicase selection** to keep specialists alive, **10% down-sampling** to cut compute, and a **refined fitness function (F_E)** that gives partial credit instead of pass/fail. On four LLMs (GPT-4, ChatGPT, Code Llama 7B, LLaMA 3 8B) over three PSB2 problems, we improved **11 of 12 model-problem combinations**. Smaller models gain the most. GI is, increasingly, a **capability amplifier** for cheap models.
{{< /summary-box >}}

## What we left on the table last time

The EuroGP 2024 paper proved the basic idea: take an LLM's buggy first draft, hand it to Grammatical Evolution, get back better code. Statistically significant gains on every model.

But the evolution itself was crude. Tournament selection. A binary fitness function. A search budget that scaled badly with the number of test cases. We had a working pipeline that was leaving wins on the floor.

This paper is the audit. We rebuilt three pieces of the GI loop — selection, sampling, fitness — and asked whether smarter evolution buys you more.

The short answer: yes.

![Enhanced pipeline: lexicase + down-sampling + F_E fitness on top of LLM-seeded GI](/images/sncs2025-exploring-gi-effect/pipeline.png)

## Why tournament selection is the wrong default

Tournament selection picks individuals by mini-competitions: grab a few at random, keep the best. It's fast and easy and it has a known weakness — **it loves generalists and kills specialists**.

That matters for code. Imagine two variants of an LLM's draft:

- Variant A: passes 60% of test cases, fails the rest mediocrely.
- Variant B: aces every integer-related test case, fails on string handling.

Variant A wins the tournament every time. Variant B carries valuable partial knowledge that crossover could have combined with another specialist on string handling — but it never makes it past round one.

Tournament selection treats program improvement like a single dimension. Real programs fail along *many* dimensions at once.

## Lexicase: keeping the weirdos alive

**Lexicase selection** evaluates candidates one test case at a time, in a random order, filtering out anyone who isn't tied for the best on that case. The order is shuffled every selection event, so being a specialist on *any* subset of cases is a survival strategy.

This sounds expensive — and on 1,000 test cases per problem, it would be. So we paired it with **down-sampling**: at every generation, only 10% of test cases are used. Different 10% each generation, so the full test suite still exerts pressure over time, just spread out.

The combination keeps specialists alive without the compute bill of full lexicase on full test suites.

## Giving the search a finer compass

The original fitness function was the fraction of test cases passed. Binary per case. A test expecting `[1, 2, 3, 4, 5]` rewards `[1, 2, 3, 4, 6]` (one digit off) the same as `"hello"` (semantic chaos).

That's wasted gradient information. We built **F_E**, a fitness function that measures *how close* the output is to the expected one, per test case. For numbers, distance. For sequences, element-wise comparison. Now "almost right" is a different number from "completely wrong," and the search can climb the right hill instead of treating the whole landscape as a cliff.

## What we ran

Four LLMs spanning the spectrum: GPT-4, ChatGPT, Code Llama 7B, LLaMA 3 8B. Three PSB2 problems chosen for difficulty diversity. Population of 200 individuals (down from 1,000 — better selection means we don't need brute force), up to 100 generations, 30 repeats each for statistical robustness.

## What we got

**11 out of 12 model-problem combinations improved.** That's not luck.

Some details worth highlighting:

- **Smaller models gained the most, again.** Code Llama 7B and LLaMA 3 8B saw the biggest relative jumps. GPT-4 also gained, but in absolute terms its starting point was already strong.
- **Lexicase actually maintains diversity.** We could see it in the population dynamics — multiple distinct specialists co-existing for many generations, recombining through crossover into hybrids that neither tournament-selection nor self-correction would ever discover.
- **Down-sampling is essentially free.** Cutting evaluations to 10% of test cases per generation didn't degrade final solution quality on our problems. This matters: GI is much more deployable when the per-generation cost is bearable.
- **F_E pays off most on hard problems.** When the LLM's seed is already close, partial credit and binary credit converge. When the seed is far, F_E gives the search something to follow.

## And once again, GI beats self-correction

We re-ran the comparison with self-correction. Same conclusion as last year, with stronger evidence: **the evolutionary loop finds fixes that the model can't find by re-prompting itself**, especially when the original code has structural issues the model is blind to.

If you're already running self-correction in production, this isn't a replacement — it's a stack. Self-correct first if you want; then run GI on top. The two failure modes are different, and the gains compound.

## What this confirms

The big-picture lesson hasn't changed since EuroGP 2024, but it's getting more solid: **GI is a capability amplifier**. It compresses the gap between cheap models and expensive ones. A 7B parameter model with a smart GI loop on top can land in the same neighborhood as a frontier model running raw — at a fraction of the inference cost.

For organizations that can't afford to call GPT-4 on every code generation request, that's not a footnote. That's the headline.

## What's still hard

Three honest limitations:

- **Oracle dependency.** GI needs a fitness signal. No test cases? You're stuck. Generating tests automatically is a separate hard problem.
- **Scale.** PSB2 is small programs. We don't yet know how this behaves on multi-file repository changes.
- **Grammar bias.** Building the mutation grammar from the LLM's output means the grammar inherits the LLM's blind spots. If the model never produces a `while` loop, the search will never explore one either.

These are the next things we're chasing.

---

### Reference

This post is a divulgative summary of:

> Pinna, G., Manzoni, L., De Lorenzo, A., Castelli, M. (2025). *Exploring the Effect of Genetic Improvement for Large Language Models generated Code*. **SN Computer Science**, 6(7).
>
> [Read the original paper (PDF)](/images/sncs2025-exploring-gi-effect/2024_SNCS_Exploring_the_Effect_of_Genetic_Improvement_for_Large_Language_Models_generated_Code.pdf)

*Research conducted at the University of Trieste and NOVA Information Management School (NOVA IMS), Universidade Nova de Lisboa.*
