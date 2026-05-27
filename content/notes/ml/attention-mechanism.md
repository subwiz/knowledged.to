---
title: Attention Mechanism
description: Explanation of the attention mechanism in AI, including self-attention, cross-attention, multi-head attention, and the mathematical formulation behind Transformers
tags:
  - notes
date: "2026-04-06T22:18:46+05:30"
lastmod: "2026-04-06T22:18:46+05:30"
draft: false
---
## Attention in AI

**Attention** is a mechanism that allows a model to focus on the most relevant parts of its input when producing an output — much like how humans pay more attention to certain words or objects in a scene than others.

### The Core Idea

Instead of treating all parts of the input equally, attention assigns **weights** to different elements, so the model can dynamically decide what's important for each step of its task.

### A Simple Example

Consider translating: *"The cat sat on the mat"* → French.

When generating the word for *"cat"*, the model should focus heavily on "cat" and less on "mat". Attention lets it do exactly that.

### How It Works (Self-Attention)

For each token (word/piece) in a sequence, attention computes three vectors:

- **Query (Q)** — "What am I looking for?"
- **Key (K)** — "What do I contain?"
- **Value (V)** — "What information do I provide?"

The attention score between tokens is computed as:

> **Attention(Q, K, V) = softmax(QKᵀ / √d) · V**

This produces a weighted sum of values, where tokens most relevant to each other get higher weights.

### Types of Attention

| Type | Description |
|---|---|
| **Self-attention** | Each token attends to all other tokens in the *same* sequence |
| **Cross-attention** | Tokens in one sequence attend to tokens in *another* (e.g., encoder → decoder) |
| **Multi-head attention** | Multiple attention operations run in parallel, each learning different relationships |
| **Causal/masked attention** | Tokens can only attend to *previous* tokens (used in GPT-style models) |

### Why It Matters

Attention is the foundation of the **Transformer architecture** (introduced in the 2017 paper *"Attention Is All You Need"*), which powers virtually all modern AI language models — GPT, Claude, Gemini, BERT, etc.

Before attention, models like RNNs had to compress entire sequences into a single vector, losing long-range context. Attention solved this by letting every token directly relate to every other token, regardless of distance.

### In Short

Attention = **a learned, dynamic weighting system** that tells the model *where to look* and *what matters most* at each step of reasoning or generation.