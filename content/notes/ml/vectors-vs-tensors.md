---
title: Vectors vs Tensors
description: Explains how vectors relate to tensors in ML, including rank, framework terminology, and KV cache shapes.
tags:
  - tensors
  - vectors
  - terminology
  - fundamentals
  - deep-learning
  - kv-cache
date: "2026-05-21T15:49:59.59372Z"
lastmod: "2026-05-21T21:20:38+05:30"
draft: false
---

Short answer: related but not identical. A vector is a special case of a tensor.

## The math hierarchy

| Term | Rank | Shape example |
|---|---|---|
| Scalar | 0 | a single number |
| Vector | 1 | `[d]` — a 1D array |
| Matrix | 2 | `[m, n]` — a 2D array |
| Tensor | N | `[d1, d2, ..., dN]` — generic N-dimensional array |

Every vector is a tensor (specifically, a rank-1 tensor). Not every tensor is a vector.

## Why the terminology blurs

In deep learning frameworks (PyTorch, JAX, TensorFlow), **everything is called a "tensor"** by convention — even scalars and vectors — because that's the underlying data type the framework operates on. That's a major reason the words get used interchangeably in ML writing.

## In the KV cache context specifically

Both terms apply at different zoom levels:

- **Per-token, per-head, per-layer**: the key and value really are **vectors** — typically 64 or 128 numbers each (the head dimension). That's why a lot of content reaches for "vector" — it's accurate at that grain and more intuitive.
- **The full cached structure**: has shape roughly `[batch, num_layers, num_heads, sequence_length, head_dim]`. That's a rank-5 **tensor**. When you "load the cache," you're loading the whole multi-dimensional block, not a single vector — so "tensor" is more precise for the stored artifact.

## Analogy

- Saying "loading vectors" is like describing a spreadsheet as "loading numbers." Technically true (every cell is a number), but it understates the structure.
- Saying "loading tensors" is like saying "loading the spreadsheet" — captures the actual shape of the thing.

## TL;DR

- Per-token K and V → **vectors**
- The full KV cache block → **tensor**
- In framework code → everything is called a `tensor` regardless of rank
- In casual technical writing → often used interchangeably, and that's usually fine
