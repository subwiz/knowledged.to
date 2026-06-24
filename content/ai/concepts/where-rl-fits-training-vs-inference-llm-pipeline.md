---
title: 'Where RL Fits: Training vs. Inference in the LLM Pipeline'
description: Explains that RL in LLMs is a training/alignment stage, not inference, with pipeline context.
tags:
  - ml
  - reinforcement-learning
  - rl
  - rlhf
  - training
  - inference
  - sft
date: "2026-06-24T05:43:38Z"
lastmod: "2026-06-24T05:59:20Z"
draft: false
---

Reinforcement Learning (RL) is fundamentally a **training** procedure, not an inference one.

Key distinction: separate the *RL training loop* from the *deployed model*. During RL, the agent **is** the model being trained, and the loop (act → get reward → update weights via gradients) *is* the learning. The game-playing example is RL happening during training: AlphaGo's policy network had its weights updated from millions of self-play games before it ever faced Lee Sedol.

## Where RL fits in the LLM pipeline

- **Pre-training** — NOT RL. Self-supervised next-token prediction over a huge corpus. No rewards, no agent loop.
- **Fine-tuning (SFT)** — NOT RL. Supervised learning on curated input/output pairs.
- **RLHF / RLVR / GRPO / PPO / DPO** — this IS the RL phase. Typically comes *after* SFT. The model is the agent: it generates responses (actions), a reward model or verifier scores them (reward), and policy weights are updated to favor higher-reward outputs.
  - **DPO** is an outlier: it skips the explicit reward-model loop and optimizes a related objective directly, but still occupies the same "alignment after SFT" slot.
- **Inference** — NO RL. Weights are frozen; the model just generates. Nothing is learned or rewarded.

So RL is one specific training stage, sandwiched between SFT and deployment. The "agent earning rewards" picture describes what happens *during* that stage, not at inference. The analogy can feel inference-flavored because the agent is "playing" (generating actions) — but in training that playing is immediately followed by a weight update, which is what makes it learning rather than mere inference.

## Nuance

There's an emerging line of work on **inference-time / test-time RL**, where a model adapts during deployment. This is a research frontier, not part of the standard pipeline.

## Abbrevations Expanded: RLHF / RLVR / GRPO / PPO / DPO

All five are RL-based methods for aligning or improving LLMs, mostly applied after SFT. Here's each:

**RLHF — Reinforcement Learning from Human Feedback.** The classic post-training alignment method. Humans rank model outputs (this response is better than that one), those rankings train a *reward model* that predicts human preference, and then an RL algorithm (usually PPO) updates the LLM to maximize that reward model's score. This is what turned raw GPT-style base models into helpful assistants.

**RLVR — Reinforcement Learning from Verifiable Rewards.** Instead of a learned reward model approximating fuzzy human taste, the reward comes from an *objective verifier*: did the math answer match the known solution, did the code pass the unit tests, did the proof check out. Because the signal is ground-truth rather than a prediction, it's far less gameable, which is why it's central to recent reasoning models on math and coding.

**PPO — Proximal Policy Optimization.** Not LLM-specific; it's a general-purpose RL algorithm (from OpenAI, 2017) and the workhorse *optimizer* inside classic RLHF. Its trick is constraining each update so the new policy can't drift too far from the old one in a single step, which keeps training stable. The "proximal" refers to that stay-close-to-the-previous-policy constraint.

**GRPO — Group Relative Policy Optimization.** A newer, lighter variant of PPO (popularized by DeepSeek). PPO needs a separate "critic" network to estimate how good each state is, which is expensive. GRPO drops the critic: it samples a *group* of responses to the same prompt, scores them all, and uses the group's average as the baseline to judge whether each response was better or worse than typical. Cheaper and simpler, and it pairs naturally with RLVR-style rewards.

**DPO — Direct Preference Optimization.** The outlier I flagged earlier. It targets the same goal as RLHF (match human preferences) but skips the RL loop entirely — no separate reward model, no sampling, no PPO. Through a mathematical reframing, it converts preference data directly into a simple supervised-style loss on the model itself. Much easier to implement and stabilize, which is why it got popular fast, though some work argues full RLHF/RLVR still wins on the hardest tasks.

A rough way to hold it together: RLHF and RLVR describe *where the reward comes from* (humans vs. verifiers); PPO and GRPO are *the optimization algorithms* that consume that reward; and DPO is an *alternative path* that sidesteps the whole reward-then-optimize machinery.