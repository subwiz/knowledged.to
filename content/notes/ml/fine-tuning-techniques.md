---
title: Fine-Tuning Techniques for LLMs
description: Comprehensive guide to LLM fine-tuning methods including full, parameter-efficient, and preference-based approaches with modern recipes and tools like LoRA and DPO
tags:
  - ml
  - llm
  - fine-tuning
  - lora
  - rlhf
  - dpo
  - grpo
  - peft
date: "2026-04-25T15:53:49.012049Z"
lastmod: "2026-04-25T21:31:15+05:30"
draft: false
---

Fine-tuning techniques can be grouped along a few axes: **what you optimize** (full weights vs. small additions), **what signal you train on** (labels, instructions, preferences, rewards), and **how the data is generated** (human, synthetic, AI-judged).

## Full Fine-Tuning (FFT)

Update every parameter in the model on a target dataset. Highest capacity, but expensive in memory and prone to catastrophic forgetting. Mostly reserved for smaller models or when you have lots of high-quality data and compute.

## Supervised Fine-Tuning (SFT) / Instruction Tuning

Train on `(prompt, ideal_response)` pairs with standard cross-entropy. "Instruction tuning" is just SFT where the dataset is a mix of tasks phrased as instructions (FLAN, Alpaca, etc.). This is almost always step one in the post-training pipeline before any preference work.

## Continued Pretraining / Domain-Adaptive Pretraining

Same objective as pretraining (next-token prediction) but on a domain corpus — code, legal, medical, your company's docs. Done before SFT when you need to inject knowledge rather than behavior.

## Parameter-Efficient Fine-Tuning (PEFT)

Freeze the base model, train a small number of new or selected parameters. The dominant family in practice:

- **LoRA** — inject trainable low-rank matrices `A` and `B` such that `ΔW = BA` is added to frozen weights. Trains <1% of parameters with near-FFT quality.
- **QLoRA** — load the base in 4-bit (NF4) and train LoRA adapters on top. Lets you fine-tune 65B+ models on a single GPU.
- **DoRA** (Weight-Decomposed LoRA) — decomposes weights into magnitude and direction, applies LoRA only to direction. Closes more of the gap to FFT.
- **Adapters** (Houlsby, Pfeiffer) — small bottleneck MLPs inserted between transformer layers. Predates LoRA; LoRA mostly replaced it because it adds no inference latency.
- **Prefix Tuning** — prepend trainable vectors to the keys/values at every layer.
- **Prompt Tuning / Soft Prompts** — only learn embeddings prepended to the input; cheapest, weakest.
- **P-Tuning v2** — prefix tuning generalized across all layers, competitive with FFT on many tasks.
- **IA³** — learn three vectors per layer that rescale keys, values, and FFN activations. Even fewer parameters than LoRA.
- **BitFit** — train only the bias terms. Surprisingly decent baseline.

## Preference / Alignment Fine-Tuning

After SFT, you align the model to preferences over responses:

- **RLHF (PPO)** — train a reward model on human-ranked pairs, then use PPO to maximize that reward with a KL penalty against the SFT model. The original ChatGPT recipe. Complex, unstable, expensive.
- **DPO (Direct Preference Optimization)** — skips the reward model entirely; derives a closed-form loss directly on preference pairs. Much simpler and now the default for most open-source alignment.
- **IPO, KTO, ORPO, SimPO** — variants that fix specific DPO failure modes (overfitting, needing paired data, requiring a separate SFT stage, etc.). ORPO is notable for combining SFT and preference learning into one stage.
- **GRPO (Group Relative Policy Optimization)** — drops the value/critic network of PPO; instead samples a group of completions per prompt and uses their relative rewards as advantages. Used in DeepSeek-R1. Memory-efficient and works well when you have a verifiable reward signal.
- **RLAIF / Constitutional AI** — same loop as RLHF, but preferences come from an AI judge guided by a written constitution rather than human labelers.

## Reasoning / Verifiable-Reward Fine-Tuning

The newer branch (DeepSeek-R1, OpenAI o-series style). Use RL (often GRPO) with **rule-based, verifiable rewards** — does the math answer match? does the code pass the tests? — to elicit long chain-of-thought without needing a learned reward model. Can be combined with distilling the resulting reasoning traces into smaller models.

## Knowledge Distillation

Train a student to match a teacher's outputs. In modern LLM practice this usually means SFT on the teacher's generated responses (and increasingly its reasoning traces), rather than the original logit-matching formulation. The DeepSeek-R1 distilled variants are the canonical recent example.

## Multi-task and Mixture Fine-Tuning

Train on a mixture of tasks/datasets simultaneously, often with task-specific prompt templates. T0, FLAN-T5, and most modern instruction-tuned models do this. Helps generalization but requires careful data balancing.

## Typical Modern Recipe

For a chat model: **pretraining → continued pretraining (optional) → SFT → DPO** (or RLHF, or GRPO if reasoning-focused), with LoRA/QLoRA used at any stage where you want to save compute.
