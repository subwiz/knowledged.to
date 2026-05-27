---
title: Chain of Thought (CoT)
description: Prompting technique where an AI model is guided — or learns — to reason through a problem step by step before arriving at the final answer, rather than jumping straight to the conclusion.
tags:
  - ml
  - ai
  - prompting
  - llm
  - reasoning
date: "2026-04-23T15:53:32.82242Z"
lastmod: "2026-04-23T21:30:11+05:30"
draft: false
---
# Chain of Thought (CoT)

Chain of Thought is a prompting technique where an AI model is guided — or learns — to reason through a problem step by step before arriving at a final answer, rather than jumping straight to the conclusion.

The core idea is that breaking down complex reasoning into intermediate steps leads to more accurate and reliable outputs, much like how a person might work through a math problem by showing their work.

## Two Main Flavors

**Explicit CoT (prompted)** — You instruct the model to "think step by step." For example:

> *"Q: If a train travels 60 mph for 2 hours, how far does it go? Let's think step by step."*
> *"A: Speed is 60 mph. Time is 2 hours. Distance = speed × time = 60 × 2 = 120 miles."*

**Implicit CoT (trained)** — Models like OpenAI's o1/o3 or Anthropic's Claude are trained to reason internally before producing a final answer, often without the user seeing the scratchpad.

## Why It Works

- Forces the model to decompose problems rather than guess
- Reduces errors on multi-step tasks (math, logic, coding)
- Makes reasoning auditable — you can see *where* it went wrong
- Particularly powerful for tasks that require planning or sequential decisions

## Where It Matters Most

Math word problems, logical puzzles, code debugging, multi-hop question answering, and anything requiring more than a one-shot lookup.

## Key Insight

One of the most impactful ideas in modern LLM prompting. Has influenced how newer models are trained at a fundamental level — the "thinking" tokens that models produce before answering is a direct descendant of this idea.
