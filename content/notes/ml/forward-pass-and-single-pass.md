---
title: Forward Pass and Single Pass in LLMs
description: Explanation of forward pass and single pass in LLMs, how transformer computation flows from embedding to output logits, and how speculative decoding exploits transformer parallelism to reduce large-model forward passes
tags:
  - notes
date: "2026-04-16T21:19:49+05:30"
lastmod: "2026-04-16T21:19:49+05:30"
draft: false
---
# Forward Pass and Single Pass in LLMs

These terms are fundamental to understanding how LLMs work under the hood.

## Forward Pass

A forward pass is a single run of data through a neural network, from input to output. In an LLM, it means feeding a sequence of tokens into the model and computing a probability distribution over the vocabulary for the next token (or all token positions simultaneously).

Here's what actually happens during a forward pass in a transformer:

1. **Embedding** — each input token is converted to a high-dimensional vector
2. **Attention layers** — each token attends to every other token in the sequence, computing relationships (this is the expensive part, O(n²) in sequence length)
3. **Feed-forward layers** — each token's representation is transformed independently through a series of matrix multiplications
4. **Output projection** — the final hidden state is projected onto the vocabulary (50K+ tokens) to produce logits (raw scores)
5. **Softmax** — logits are converted to probabilities, and you sample or argmax to pick the next token

The cost of one forward pass is dominated by **loading the model weights from GPU memory** (HBM). A 70B parameter model at FP16 is ~140GB of weights, and you need to stream all of those through the GPU's compute cores for every single pass. This is why inference is memory-bandwidth-bound.

## Single Pass

"Single pass" means doing the forward pass exactly once for a given input — you feed in your tokens, run through the entire network once, and get your output logits back. It's contrasted with iterative or multi-step processes that would require multiple network executions.

## How These Connect to Speculative Decoding

Normal autoregressive generation works like this:

```
Input: "The capital of France"
Pass 1 → predicts "is"
Pass 2 → predicts "Paris"
Pass 3 → predicts "."
```

Three separate forward passes, each loading all model weights. Each pass is largely serial — you can't start pass 2 until you have the token from pass 1.

Speculative decoding breaks this seriality. When the large model runs its forward pass to *verify* the draft model's candidates, it processes **all candidate positions in parallel** in that single pass. Here's why that's possible:

Transformers are inherently parallel across the sequence dimension during a forward pass. Given the sequence `["The", "capital", "of", "France", "is", "Paris", "."]`, the model can compute the probability of every token given its predecessors simultaneously in one shot — that's how training works. Speculative decoding borrows this property for inference:

```
Draft model generates: ["is", "Paris", "."]  (3 separate small passes)

Large model gets:  "The capital of France" + ["is", "Paris", "."]
                   → verifies all 3 draft tokens in ONE forward pass
                   → accepts "is" ✓, accepts "Paris" ✓, accepts "." ✓
```

So instead of 3 expensive large-model passes, you do 1. The memory bandwidth cost of that one pass is nearly the same whether you're verifying 1 token or 8, because the weight-loading dominates — not the arithmetic on token positions.

## The KV Cache Wrinkle

Modern LLMs use a **KV cache** (key-value cache) to avoid recomputing attention for tokens already processed. Each forward pass only computes attention for *new* tokens against the cached representations of prior tokens. This makes each incremental generation step cheaper than a full forward pass, but the fundamental bottleneck — streaming weights from memory — remains the same. Speculative decoding's gains hold regardless.

## Key Takeaway

A forward pass is the atomic unit of computation in a neural network. Speculative decoding's trick is turning what would be N serial forward passes on the large model into 1, by exploiting the parallelism that transformers already have built in.