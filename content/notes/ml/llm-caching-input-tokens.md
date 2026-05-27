---
title: Why LLM Caching Is Only for Input Tokens
description: Explains why LLM prompt caching applies to reusable input-token prefill, not sequential output decoding.
tags:
  - caching
  - inference
  - llm
  - kv-cache
  - prompt-caching
  - transformers
date: "2026-05-21T15:43:26.86581Z"
lastmod: "2026-05-21T21:14:09+05:30"
draft: false
---

Why prompt caching applies to inputs and not outputs in LLM APIs (Anthropic, OpenAI, Google). The asymmetry comes down to how inputs vs. outputs are computed, and what's actually reusable across requests.

## Inputs are processed in parallel; outputs are generated sequentially

When a prompt comes in, the transformer computes KV (key/value) tensors for every token in one forward pass — the **prefill** phase. Those KV tensors are a deterministic function of the input, so they can be stashed and reused if the same prefix shows up again.

Output generation is the **decode** phase: one token at a time, each step depending on the sampled previous token. Even with the same input, sampling (temperature, top-p) can produce different sequences. There's nothing stable to cache across requests.

## Inputs are reused; outputs typically aren't

Prompt caching exists because long system prompts, RAG contexts, code repos, and document attachments get sent again and again across calls. Caching them turns expensive recomputation into a lookup.

Outputs, by contrast, are consumed once by the user and rarely sent back verbatim. Caching a 2000-token answer to reuse it would require a future request to ask for that exact answer — which almost never happens. When it does, that's response caching at the application layer (e.g. memoizing a deterministic query), not model-level caching.

## But outputs DO get cached — just on the next turn

In a multi-turn conversation, yesterday's assistant message becomes part of today's input. At that point, the provider's prompt cache will reuse those tokens.

So there isn't really a "no caching for outputs" rule. It's more that output tokens only become cacheable once they've transitioned into being **input** tokens for a subsequent call. Anthropic, OpenAI, and Google all do this implicitly when you replay a conversation.

## The KV cache during decode is a separate thing

Inside a single generation, each newly produced token's KV vectors get appended to a running cache so the next token doesn't have to re-attend over the whole sequence from scratch. That's a **within-request** optimization and is universal.

It's not what people mean by "prompt caching." The cross-request prompt cache that providers charge a discount for is specifically about the prefill phase being skippable.

## Pricing reflects the compute asymmetry

- **Output tokens** cost more than input tokens (often 4–5×) because each one requires a full forward pass through the model with no parallelism.
- **Cached input tokens** cost even less than uncached input — you're skipping the prefill compute entirely and just loading tensors.
- There's no equivalent shortcut for output generation. You can't "skip" producing a token you haven't produced yet.

## TL;DR

| | Input | Output |
|---|---|---|
| Computation | Parallel prefill | Sequential decode |
| Determinism | Deterministic given input | Stochastic (sampling) |
| Reuse pattern | Same prompts sent repeatedly | Generated once, rarely resent |
| Cacheable across requests? | Yes | Not until it becomes input on the next turn |

Inputs are deterministic, parallelizable, and frequently reused — perfect cache candidates. Outputs are sequential, stochastic, and consumed once. The moment they're *not* consumed once, they've become inputs anyway.
