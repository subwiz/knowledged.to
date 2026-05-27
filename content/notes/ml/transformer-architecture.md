---
title: Transformer Architecture
description: Overview of the Transformer architecture including self-attention, multi-head attention, positional encoding, encoder-decoder structure, and key variants like BERT, GPT, and T5
tags:
  - notes
date: "2026-04-06T22:15:42+05:30"
lastmod: "2026-04-06T22:15:42+05:30"
draft: false
---
The Transformer architecture, introduced in the 2017 paper *"Attention Is All You Need"* by Vaswani et al., revolutionized AI by replacing recurrent networks with a purely attention-based design. Here's a breakdown of how it works:

## Core Idea: Self-Attention

Instead of processing sequences step-by-step (like RNNs), Transformers process all tokens in parallel and learn *relationships between every pair of tokens* simultaneously. This is done via **self-attention**.

For each token, three vectors are computed:
- **Query (Q)** — what this token is looking for
- **Key (K)** — what this token offers
- **Value (V)** — the actual content to pass forward

Attention is computed as: `Attention(Q, K, V) = softmax(QKᵀ / √d_k) · V`

The scaling by `√d_k` prevents dot products from growing too large in high dimensions.

## Multi-Head Attention

Rather than one attention computation, the model runs **multiple attention heads in parallel** — each learning different types of relationships (e.g., syntactic, semantic, co-reference). Their outputs are concatenated and projected.

## Positional Encoding

Since there's no recurrence, the model has no inherent sense of order. **Positional encodings** (sinusoidal functions or learned embeddings) are added to token embeddings to inject sequence position information.

## The Encoder-Decoder Structure

The original Transformer had two stacks:

**Encoder** (used in models like BERT):
1. Input embeddings + positional encoding
2. Multi-head self-attention
3. Feed-forward network (FFN)
4. Layer norm + residual connections around each sub-layer

**Decoder** (used in models like GPT):
1. Same as encoder, but adds **masked** self-attention (tokens can only attend to past tokens)
2. **Cross-attention** layer — attends to the encoder's output
3. FFN + layer norm + residuals

Modern LLMs like GPT are **decoder-only**; models like BERT are **encoder-only**.

## Feed-Forward Network (FFN)

After attention, each position passes through a small 2-layer MLP independently:
`FFN(x) = max(0, xW₁ + b₁)W₂ + b₂`

This adds non-linearity and expands the model's representational capacity.

## Why Transformers Won

| Property | RNNs/LSTMs | Transformers |
|---|---|---|
| Parallelism | Sequential | Fully parallel |
| Long-range dependencies | Struggles | Handles natively |
| Training speed | Slow | Fast (on GPUs/TPUs) |
| Scalability | Limited | Scales to billions of params |

## Key Variants

- **BERT** — Encoder-only, trained with masked language modeling; great for classification and understanding tasks.
- **GPT** — Decoder-only, trained autoregressively; great for generation.
- **T5 / BART** — Full encoder-decoder; great for seq2seq tasks like translation and summarization.
- **Vision Transformer (ViT)** — Applies the same architecture to image patches instead of text tokens.

The Transformer's combination of parallelism, expressiveness, and scalability is what enabled the modern LLM era.