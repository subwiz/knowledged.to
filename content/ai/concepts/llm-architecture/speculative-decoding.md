---
title: What is Speculative Decoding?
description: Explains speculative decoding, which pairs a small draft model with a large target model to accelerate LLM inference without changing outputs.
tags:
  - llm
  - inference
  - speculative-decoding
  - optimization
date: "2026-06-24T17:18:08Z"
lastmod: "2026-06-24T17:18:44Z"
draft: false
---

Speculative decoding is a technique to make LLM text generation faster without changing the output.

## The problem it solves

Normally a large model generates one token at a time, and each token requires a full forward pass through the whole network. That's slow, and it's bottlenecked by memory bandwidth rather than raw compute — the hardware is often idle waiting to load the model's weights.

## The trick

Pair the large "target" model with a small, fast "draft" model:

1. The draft model quickly guesses the next several tokens (e.g. 4–5) in a row.
2. The large model checks all of those guesses in a single forward pass, because verifying tokens in parallel is cheap — the same memory load whether scoring one token or several.
3. Wherever the draft's guess matches what the large model would have produced, you keep it for free.
4. At the first mismatch, discard the rest; the large model supplies the correct token itself.

## Key property

The output is mathematically identical to what the large model would produce on its own. The draft only proposes; the large model has final say on every token. Result: a speedup (often 2–3x) with zero quality loss.

## What makes or breaks it

- The draft model must be much cheaper than the target.
- Its guesses must be accurate often — if the draft is usually wrong, you throw away its work and gain nothing. This is why drafts are typically a tiny version of the same model family, so predictions align.

## Caveat

Verification needs the large model's raw probability distribution (logits), so speculative decoding only works when you control both models locally. You can't use a hosted frontier API as the verifier, because those APIs don't expose the internals it requires.