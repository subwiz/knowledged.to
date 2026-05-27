---
title: 'LLM Prompt Caching: Implicit vs Explicit'
description: Explains implicit vs explicit LLM prompt caching, prefix constraints, provider support, and when to use each.
tags:
  - llm
  - caching
  - inference
  - kv-cache
  - prompt-engineering
  - gemini
  - anthropic
  - openai
date: "2026-05-21T16:08:55.751098Z"
lastmod: "2026-05-21T21:39:32+05:30"
draft: false
---

Caching in LLM inference is about reusing the **KV-cache** computed from a prompt prefix so the model doesn't re-process the same tokens on every request. The "implicit vs explicit" distinction is about *who manages that cache*.

## Prompt Prefix: The Underlying Mechanism

"Prefix" means literally the starting tokens of the prompt — the bytes from position 0 onward, in order, that two requests have in common before they diverge.

When a transformer processes a prompt, it computes attention keys and values for each token. The KV state for token N depends on every token before it. So if request A is `[system prompt][doc X][question 1]` and request B is `[system prompt][doc X][question 2]`, the KV state for `[system prompt][doc X]` is identical in both — the model can skip recomputing it and pick up at the divergence point.

**Key constraint:** it has to be a true prefix — byte-identical, from token zero. If request B differs by even one token (a different system prompt, an extra space, a swapped ordering), the cache is invalidated from that point onward, because every downstream token's attention now depends on different upstream state. You cannot cache the *middle* or *end* of a prompt while changing the beginning.

### Prompt ordering for cacheability

Structure prompts so volatile content comes last:

1. System prompt (stable across all calls)
2. Tool/function definitions (stable per app version)
3. Large static context — RAG documents, knowledge base chunks (stable per session)
4. Conversation history (grows but is append-only, so prior turns stay prefix-stable)
5. The new user message (the only volatile part)

Flip that order — put the user message first — and the cache busts on every turn.

## Implicit Caching

Automatic. The provider's inference layer detects when a new request shares a long prefix with a recent one and silently reuses the cached state. No code changes — just send the same system prompt or document context at the start of each request, and if the platform's heuristics fire (usually requiring a minimum prefix length and a short time since the last hit), you get a discount on the cached tokens.

**Providers:** Gemini, OpenAI, and DeepSeek all do versions of this.

**Pros:** Zero integration effort.

**Cons:** Best-effort. No TTL you control, no guarantee a given request will hit, and cold-start requests pay full price.

## Explicit Caching

You tell the provider: "store this exact context, give me back a handle, and bill me for storage until it expires." Subsequent requests reference the handle instead of resending the content.

**Providers:**
- Anthropic — `cache_control: {type: "ephemeral"}` markers on message blocks
- Gemini — `CachedContent` API

**Pros:** Deterministic. Within the TTL, you *will* hit. Storage fee + one-time write cost, but reads are dramatically cheaper than re-tokenizing.

**Cons:** Requires integration code; you pay storage cost and must manage cache lifecycle (creation, TTL, invalidation).

## When to Use Which

- **Implicit** wins for workloads with naturally repeating prefixes and bursty traffic where you can't easily reason about cache lifetime — chatbots, IDE autocomplete, anything where a system prompt is shared across many short-lived calls.
- **Explicit** wins when you have a large, stable context (long document, big tool/schema block, knowledge base chunk) queried many times over a known window. Trade integration code + storage cost for guaranteed savings.

In practice teams often **layer them** — explicit caches for the heavy stable stuff (system prompt + tool definitions + large RAG context), implicit caching catching the rest opportunistically.
