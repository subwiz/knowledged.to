---
title: Attention in Machine Learning
description: Explanation of the attention mechanism in ML, covering Query/Key/Value, self-attention, multi-head, causal, cross-attention, and efficiency variants like FlashAttention and GQA.
tags:
  - ml
  - ai
  - attention
  - transformers
  - self-attention
  - deep-learning
date: "2026-05-17T05:54:45.17871Z"
lastmod: "2026-05-17T11:25:39+05:30"
draft: false
---

Attention is a mechanism that lets a model dynamically decide *which parts of the input matter most* when producing each piece of output. Instead of compressing everything into one fixed representation, the model computes a weighted combination of inputs where the weights are learned and depend on context.

## Intuition

When translating "the cat sat on the mat" to French, generating the word for "cat" should mostly pay attention to "cat" in the source — not "mat" or "on." Attention makes this routing explicit and differentiable.

Before attention (Bahdanau et al., 2014, in neural machine translation), encoder-decoder RNNs had to squeeze the whole source sentence into a single hidden vector, which broke down on longer inputs.

## Mechanics: Query, Key, Value

The standard formulation is **scaled dot-product attention**:

- Each input position produces three vectors via learned linear projections: a **query** (Q), a **key** (K), and a **value** (V).
- For a given query, compute its dot product with every key. This gives a similarity score: how relevant is each position to what I'm currently looking for?
- Scale by √d_k (to keep gradients stable under large d_k) and apply softmax to turn scores into a probability distribution.
- Take the weighted sum of the values using those probabilities.

In one line:

```
Attention(Q, K, V) = softmax(QKᵀ / √d_k) · V
```

## Self-attention and why it was a big deal

The Transformer (Vaswani et al., 2017, "Attention is All You Need") made *self-attention* the central operation: every token attends to every other token in the same sequence — Q, K, V all come from the same input. This unlocked two things RNNs couldn't do well:

1. **Long-range dependencies** — any position can directly reference any other in one step, instead of information having to flow through many recurrent timesteps.
2. **Parallelism** — all positions are processed simultaneously, which is why Transformers train so much faster than RNNs on GPUs.

## Important variants

- **Multi-head attention** — run several attention operations in parallel with different projections, then concatenate. Each head can specialize (one tracks syntax, another tracks coreference, etc.).
- **Causal / masked attention** — in decoders, mask out future positions so a token only attends to previous ones. This is what makes autoregressive generation possible.
- **Cross-attention** — Q comes from the decoder, K and V from the encoder. Used in seq2seq Transformers and in diffusion models for conditioning on text.
- **Efficiency variants**:
  - **FlashAttention** — memory-efficient exact attention via tiling and recomputation.
  - **Grouped-query attention (GQA) / multi-query attention (MQA)** — share K, V across heads to shrink the KV cache.
  - **Sliding-window / sparse attention** — for long contexts where full O(n²) attention is too expensive.

## TL;DR

Attention is **content-based, soft, differentiable lookup**. Self-attention applied that lookup to a sequence's own tokens. Modern LLMs are essentially scaled-up stacks of self-attention layers (plus feedforward blocks and normalization).

## References

- Bahdanau, Cho, Bengio (2014). *Neural Machine Translation by Jointly Learning to Align and Translate.*
- Vaswani et al. (2017). *Attention Is All You Need.*
- Dao et al. (2022). *FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness.*
