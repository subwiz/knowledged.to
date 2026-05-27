---
title: Speculative Decoding
description: Explanation of speculative decoding, an inference optimization that uses a fast draft model to propose tokens verified in parallel by a large model, achieving 2–3x throughput gains with identical output quality
tags:
  - notes
date: "2026-04-16T20:54:05+05:30"
lastmod: "2026-04-16T20:54:05+05:30"
draft: false
---

Speculative decoding is a clever inference optimization technique that exploits a fundamental asymmetry in how LLMs work: **verifying a token is much cheaper than generating one**.

## The Basic Setup

You run two models simultaneously — a small, fast "draft" model and your large "target" model. The draft model generates several tokens ahead in a single pass (typically 4–8 tokens). The large model then verifies all of those candidate tokens *in parallel* in one forward pass. If the draft tokens match what the large model would have produced, you accept them all at once. If a token diverges, you reject it (and everything after it) and fall back to the large model's output for that position.

## Why This Is So Effective

LLM inference is memory-bandwidth-bound, not compute-bound. The GPU spends most of its time *loading model weights* from HBM (high bandwidth memory), not doing matrix multiplications. A forward pass that verifies 8 tokens costs nearly the same memory bandwidth as verifying 1 token, so you get multiple accepted tokens for roughly the price of one. The result is a **2–3x throughput improvement** with mathematically guaranteed identical output — it's not an approximation.

## The Catch: Draft Model Quality Matters

The speedup depends entirely on how often the draft model's predictions are accepted. If the draft model diverges frequently (low acceptance rate), you're paying the overhead of running two models for minimal gain. In practice, a good draft model for a given target model has a 70–85% token acceptance rate, which is where the 2–3x gains come from.

## Variants Worth Knowing

- **Self-speculative decoding** — uses the target model itself with early exit layers as the draft, avoiding the need for a separate model
- **Medusa** — adds multiple parallel draft "heads" to a single model, predicting several tokens ahead simultaneously without a separate model
- **EAGLE / EAGLE-2** — uses a featherweight autoregressive head trained specifically to mimic the target model's distribution, achieving higher acceptance rates than standard speculative decoding
- **SpecInfer** — optimized for batched serving scenarios where multiple requests are in-flight

## When It Helps Most

Speculative decoding shines in low-batch, latency-sensitive workloads (like interactive chat or copilot features) where you can dedicate resources to a single request. In high-throughput batch scenarios, continuous batching already keeps the GPU saturated, so the gains are less pronounced.

## Practical Implementation

If you're self-hosting models with vLLM or SGLang, both support speculative decoding natively. You configure a `speculative_model` alongside your target model, and the inference engine handles the rest. For hosted APIs, some providers are now baking it in transparently — it's worth checking whether your provider supports it, as it can cut latency noticeably for streaming responses.

## Key Takeaway

Speculative decoding offers identical output quality with the same mathematical guarantees, just faster — a genuine free lunch in systems engineering. As of April 2026, it delivers 2–3x speedup and is supported natively in vLLM and SGLang.