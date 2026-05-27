---
title: GRPO — Group Relative Policy Optimization
description: Critic-free RL algorithm that replaces PPO's value model with group-relative rewards for LLM fine-tuning.
tags:
  - ai
  - rl
  - rlhf
  - grpo
  - deepseek
  - fine-tuning
date: "2026-05-19T17:17:58.420793Z"
lastmod: "2026-05-19T22:48:31+05:30"
draft: false
---

GRPO is a reinforcement learning algorithm introduced by DeepSeek (DeepSeekMath, later DeepSeek-R1) as a more efficient alternative to PPO for fine-tuning LLMs with RL.

## Core Idea

PPO needs a separate **value model (critic)** of comparable size to the policy to estimate the baseline for advantage calculation. That doubles memory and compute. GRPO ditches the critic entirely.

Instead, for each prompt it samples a **group** of G outputs from the current policy, scores each with the reward model, and uses the group's mean and standard deviation as the baseline:

```
A_i = (r_i - mean(r_1..r_G)) / std(r_1..r_G)
```

An output's "advantage" is just how much better or worse it scored than its siblings from the same prompt. Outputs above the group mean get pushed up, below get pushed down. **The relative ranking within the group is the signal.**

## Why It Matters

- **Cheaper**: no critic network → roughly half the memory footprint vs PPO.
- **Naturally suited to verifiable rewards**: for math/code where you can grade outputs with a checker, sample G attempts, grade them, let the relative scores drive learning. No need to train a value model — notoriously hard to fit for sparse rewards.
- **Stable**: keeps PPO's clipped surrogate objective and KL penalty against a reference model, so it inherits PPO's stability properties without the critic.

## Where It's Used

- **DeepSeek-R1** reasoning training is the headline use case — GRPO with rule-based rewards (correctness + format) elicited chain-of-thought without any SFT bootstrapping in the R1-Zero variant.
- Has become a common choice for RLHF/RLVR pipelines where you want PPO's behavior without the critic overhead.

## Trade-off

Leans on having a *useful* reward signal across the group. If all G samples score identically (all wrong, all right), the advantage collapses to zero and you learn nothing from that prompt. Pairs best with tasks where sampling produces meaningful score variance.

## Relationship to PPO

GRPO is essentially PPO with the critic replaced by a group-relative Monte Carlo baseline. Everything else — the clipped surrogate objective, the KL penalty against the reference model, the importance sampling ratio — is inherited from PPO.
