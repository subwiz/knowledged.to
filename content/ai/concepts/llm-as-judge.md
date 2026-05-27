---
title: LLM as Judge
description: Using a language model to evaluate another model's outputs as a scalable proxy for human preference judgments.
tags:
  - llm
  - evaluation
  - rlaif
  - judge
  - elo
  - rlhf
date: "2026-05-14T10:34:24.157662Z"
lastmod: "2026-05-14T16:04:30+05:30"
draft: false
---

Using a language model to evaluate the outputs of another model (or itself) instead of relying on humans or rigid automated metrics like BLEU/ROUGE/exact-match. Give the judge model a response (or a pair of responses) plus a rubric or question, and it returns a score, a label, or a winner.

## Why it exists

For open-ended generation — chat answers, code explanations, summaries, agent traces — string-overlap metrics don't capture quality, and human eval is slow and expensive. Once frontier LLMs got good enough, they became decent proxies for human raters on a lot of tasks, so they're now the default evaluator in MT-Bench, Chatbot Arena, G-Eval, and most internal eval pipelines.

It's also the "AI feedback" half of **RLAIF** and **Constitutional AI** — the judge produces the preference signal that would otherwise come from a human, which then feeds into DPO or GRPO. This is why it shows up next to ELO: Chatbot Arena runs pairwise LLM (and human) judgments and uses Elo to convert win-rates into a ranking.

## Common shapes

- **Single-answer grading** — judge scores one response against a rubric (1–10, pass/fail, criterion-wise).
- **Pairwise** — judge picks A or B given the same prompt. Cleaner signal, pairs well with Elo.
- **Reference-based** vs **reference-free** — with or without a gold answer to compare against.

## Known biases

- **Position bias** — favors the first option in pairwise.
- **Verbosity bias** — longer answers score higher.
- **Self-preference bias** — a model rates its own outputs higher.
- **Style over substance** — confident, well-formatted, wrong answers beat hedged correct ones.

## Mitigations

- Swap positions and average across both orderings.
- Use a judge that's *stronger* than the models being evaluated.
- Force the judge to produce chain-of-thought reasoning before the score.
- Ensemble multiple judges.
- Calibrate against a small human-labeled set so you know how much to trust the numbers.

## Mental model

LLM-as-judge is a **cheap, scalable, biased estimator of human preference**. Useful as a gradient during iteration; dangerous if you treat its absolute scores as ground truth.

## Related

- ELO scoring for AI evaluation (Chatbot Arena ranking)
- RLAIF, Constitutional AI
- DPO, GRPO (consume preference signals the judge produces)
- MT-Bench, G-Eval
