---
title: Elastic Looped Transformers (ELT)
description: Overview of Elastic Looped Transformers, an adaptive compute architecture that loops a shallow transformer block multiple times to dynamically allocate compute based on input complexity
tags:
  - notes
date: "2026-04-16T22:36:40+05:30"
lastmod: "2026-04-16T22:36:40+05:30"
draft: false
---

Elastic Looped Transformers (ELT) are a recent architectural innovation that rethinks how transformer layers are applied — moving from a fixed, one-pass stack to a dynamic, recurrent execution model.

## The Standard Transformer Problem

In a conventional transformer, you have a fixed stack of N layers (say, 96 layers in a large model). Every input always passes through all 96 layers exactly once. This is rigid in two ways:

- **Every input gets the same compute budget**, regardless of whether it's a trivial question or a complex reasoning problem.
- **Depth is fixed at architecture design time** — you can't adapt it post-training without retraining.

## The Core Idea: Looping

ELT takes a shallower set of transformer layers and runs them multiple times in a loop — hence "looped." Instead of having 96 distinct layers, you might have 12 layers that execute 8 times, with hidden states passed from one loop iteration to the next.

This makes the architecture inherently **recurrent** — information from one pass through the layer block feeds into the next, allowing the model to iteratively refine its representations. Each loop can be thought of as a "thinking step," where the model revisits the same input with an updated internal state.

## The "Elastic" Part

The elastic aspect allows the number of loop iterations to be adjusted dynamically at inference time based on the difficulty or complexity of the input:

- Simple inputs might need only 3–4 loops; complex multi-step reasoning might use 12+.
- This is called **adaptive compute** or **dynamic depth**.
- You can tune the speed/quality tradeoff at inference time without retraining.
- Easy inputs are processed cheaply, hard inputs get more compute.
- The same model checkpoint can serve both latency-sensitive and quality-sensitive use cases.

## How Hidden State Carries Over

Between loop iterations, the model's hidden state (the tensor representations at each token position) is passed forward. The model builds a progressively refined understanding of the input across iterations — similar to how a human might re-read a complex sentence before answering.

Some ELT designs incorporate a learned **halting mechanism** — a small auxiliary network that predicts whether another loop iteration would meaningfully improve the output, allowing the model to stop early when it's confident.

## Comparison to Other Approaches

| Approach | Compute per input | Adaptivity | Memory |
|---|---|---|---|
| Standard Transformer | Fixed (all layers, once) | None | KV cache only |
| Mixture of Experts | Variable (sparse routing) | Partial | Large parameter count |
| ELT | Variable (loop count) | High | Recurrent hidden state |
| Mamba/RWKV | Fixed (recurrent step) | None | Compressed state |

ELT sits in an interesting position — more adaptive than standard transformers, more memory-efficient than MoE, and more expressive than fixed-recurrent models like Mamba.

## Why It Matters for AI Engineers

- **Inference cost optimization** — dynamically allocate compute per request based on complexity rather than paying the same cost for everything.
- **Reasoning tasks** — iterative refinement makes ELT well-suited for chain-of-thought style reasoning baked into the architecture; each loop iteration is implicitly a reasoning step.
- **Edge/constrained deployment** — cap loop count for resource-constrained environments; run more iterations in the cloud — same weights, different compute budget.

## Current State (April 2026)

ELT remains primarily a research architecture — not yet adopted at mainstream transformer deployment scale. Main open challenges:
- Training stability (looped architectures can suffer from gradient issues across iterations)
- Overhead of the halting mechanism

Represents a promising direction in the broader movement toward **adaptive compute** — the idea that model intelligence should scale with problem difficulty, not be statically provisioned.