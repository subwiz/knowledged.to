---
title: The Modern LLM Training Pipeline
description: Explains the four-stage modern LLM training pipeline from pre-training through verifiable-reward RL.
tags:
  - llm
  - training
  - machine-learning
  - rlhf
  - pretraining
date: "2026-06-24T06:05:23Z"
lastmod: "2026-06-24T06:05:36Z"
draft: false
---

The modern LLM training pipeline has four main stages, each producing a more capable and aligned model from the one before.

## 1. Pre-training

The model learns language and world knowledge by predicting the next token over a massive corpus (trillions of tokens of web text, code, books). This is self-supervised — no human labels — and accounts for the vast majority of compute. The output is a "base model" that has broad knowledge but isn't good at following instructions; it just continues text. Key levers here are data quality/filtering, scale (parameters × tokens, governed by scaling laws), and architecture (a decoder-only Transformer).

## 2. Supervised fine-tuning (SFT) / instruction tuning

The base model is fine-tuned on a smaller, curated set of high-quality prompt–response pairs demonstrating desired behavior (answering questions, following instructions, formatting, refusing harmful requests). This teaches the model the *form* of being a helpful assistant. Data is often human-written or carefully filtered, and increasingly includes synthetic data generated and vetted by other models.

## 3. Preference optimization / alignment (RLHF and successors)

The model is tuned to match human preferences about *which* response is better. The classic recipe is RLHF: humans rank model outputs, a reward model is trained on those rankings, and the LLM is optimized against that reward with reinforcement learning (PPO). Lighter-weight alternatives like DPO (Direct Preference Optimization) skip the separate reward model and optimize directly on preference pairs. This stage shapes helpfulness, tone, safety, and honesty. Constitutional AI / RLAIF variants replace some human feedback with AI-generated feedback against a written set of principles.

## 4. Reasoning / RL on verifiable rewards (the newer stage)

The most recent addition trains models to reason through long chains of thought, optimized with reinforcement learning against *automatically checkable* rewards — math problems with known answers, unit-tested code, verifiable tasks. This is what produces "reasoning models" that think before answering. It scales differently because correctness can be checked programmatically rather than by humans.

## Surrounding all stages

- **Data curation** is arguably the highest-leverage work at every stage — deduplication, quality filtering, decontamination, and mixing ratios.
- **Evaluation and red-teaming** run continuously to measure capability and catch safety failures.
- **Post-training also includes** tool-use/function-calling training, long-context extension, and distillation (training a smaller model to mimic a larger one).

## Mental model

Pre-training builds raw knowledge and capability, while everything after it ("post-training") is about *eliciting and shaping* that capability into a safe, useful assistant. Most of the cost is in stage 1, but most of the *felt* quality of a modern model comes from stages 2–4.