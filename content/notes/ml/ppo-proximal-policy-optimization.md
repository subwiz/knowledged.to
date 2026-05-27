---
title: PPO — Proximal Policy Optimization
description: Overview of PPO, the clipped policy-gradient RL algorithm used in RLHF for InstructGPT and original ChatGPT.
tags:
  - ai
  - rl
  - rlhf
  - ppo
  - openai
  - fine-tuning
date: "2026-05-19T17:18:44.221551Z"
lastmod: "2026-05-19T22:49:14+05:30"
draft: false
---

PPO is a reinforcement learning algorithm from OpenAI (Schulman et al., 2017) that became the default workhorse for RLHF — it's what trained InstructGPT and the original ChatGPT.

## Core Idea

Policy gradient methods are unstable because a single large update can collapse the policy. PPO fixes this by **staying close to the previous policy on each update** — the "proximal" part. It does this with a **clipped surrogate objective**:

```
L = min(r(θ)·A, clip(r(θ), 1-ε, 1+ε)·A)
```

Where:
- `r(θ)` = probability ratio between new and old policy
- `A` = advantage (how much better an action was than the baseline)
- `ε` = clip range, typically 0.1–0.2

If the new policy tries to change the probability of an action by more than ε, the gradient gets clipped — preventing destructive updates while still allowing improvement.

## In RLHF Specifically

PPO uses **four models loaded simultaneously**:

1. **Policy** — the LLM being trained
2. **Reference** — frozen copy of the policy for the KL penalty (keeps the model from drifting too far from its SFT origin)
3. **Reward model** — scores completions
4. **Value model (critic)** — estimates expected return for advantage calculation

That fourth model is exactly what GRPO eliminates by using group-relative baselines instead.

## Why It Dominated

- Simpler than **TRPO** (its predecessor, which used a hard KL constraint via constrained optimization).
- More stable than vanilla policy gradient.
- Works well across a huge range of tasks — robotics, games, and LLM fine-tuning all use the same algorithm with minimal changes.

## Limitations

- **Memory-heavy**: four models in GPU memory at once.
- **Critic is hard to train** with sparse/delayed rewards — common in RLHF where the reward only comes at end of generation.
- **Hyperparameter-sensitive**: KL coefficient, clip range, value loss weighting all need tuning.

These limitations motivated alternatives like **DPO** (no RL at all, direct preference optimization on pairs) and **GRPO** (drops the critic).
