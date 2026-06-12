---
title: 'AdapTime: Adaptive Temporal Reasoning in LLMs'
description: Paper summary of AdapTime, an adaptive planner for temporal reasoning in LLMs.
tags:
  - llm
  - temporal-reasoning
  - agents
  - reasoning
  - acl-2026
date: "2026-06-11T15:36:12Z"
lastmod: "2026-06-11T15:36:22Z"
draft: false
---

**Paper:** AdapTime: Enabling Adaptive Temporal Reasoning in Large Language Models (arXiv 2604.24175). Accepted to ACL 2026 Findings. Authors from Xi'an Jiaotong University, City University of Hong Kong, and Tencent Jarvis Lab. Code: https://github.com/Applied-Machine-Learning-Lab/ACL2026-AdapTime

## Problem

LLMs reason well over general knowledge but are weak at temporal questions — anything hinging on time: "Who was CEO of X before Y?", multi-hop date arithmetic, event ordering, "what was true as of 2019?". Prior approaches had two flaws: they lean on external tools or manual verification (poor generalizability), and they apply one fixed pipeline to every question. A fixed pipeline wastes compute on easy questions and under-reasons on hard ones.

## Core idea: adaptivity

Instead of a static chain, an LLM planner inspects each question and decides which reasoning actions to invoke and in what order, based on the question's temporal complexity. Simple lookups get a short path; complex multi-hop temporal questions get the full treatment. No external tools required — it works on top of an off-the-shelf LLM.

## The three actions the planner orchestrates

- **Reformulate** — decomposes a complex temporal question into simpler, focused sub-questions. The heavy lifter for multi-hop queries and intricate time expressions.
- **Rewrite** — restates/normalizes the question or intermediate form so temporal constraints are explicit and unambiguous (resolving relative time references, pinning the reference point).
- **Review** — a verification/self-check step that catches temporal inconsistencies before committing to an answer.

## Results

Consistently beats strong baselines, with the biggest gains on temporally complex tasks, and robust generalizability across scenarios (the point of avoiding scenario-specific tooling).

## Why it's relevant to AI application engineering

Two transferable ideas, even outside temporal QA:

1. **Planner-decides-the-pipeline** is a clean, model-agnostic way to do adaptive-depth reasoning — spend tokens proportional to difficulty instead of a one-size chain. Directly applicable to cost/latency control in production agents.
2. **Reformulate -> rewrite -> review** is a reusable scaffold for any task where ambiguity and multi-hop decomposition cause errors — essentially a structured self-correction harness you can prompt-engineer onto an existing model without fine-tuning or tools.

## Sources

- https://arxiv.org/abs/2604.24175
- https://arxiv.org/html/2604.24175v1
- https://www.themoonlight.io/en/review/adaptime-enabling-adaptive-temporal-reasoning-in-large-language-models
