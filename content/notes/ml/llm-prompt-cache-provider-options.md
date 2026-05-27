---
title: LLM Prompt Cache Options Across Providers
description: Compares prompt/KV cache TTLs, controls, pricing, scope, and strategies across major LLM providers.
tags:
  - llm
  - caching
  - prompt-caching
  - anthropic
  - openai
  - gemini
  - bedrock
  - deepseek
  - vllm
  - semantic-caching
  - kv-cache
date: "2026-05-21T16:49:20.114554Z"
lastmod: "2026-05-21T22:20:05+05:30"
draft: false
---
# LLM Prompt Cache Options Across Providers

A reference covering cache TTL options and other cache-control dimensions across major LLM providers as of May 2026.

## TTL mechanics

### Fixed-duration TTLs
- **Anthropic**: 5-min (default) and 1-hour (extended). Cache writes cost 1.25× base input for 5-min TTL, 2× for 1-hour. Cache reads ≈ 10% of base input. TTL refreshes on each read (sliding window).
- **AWS Bedrock**: 5-min default, 1-hour added Jan 2026 for Claude Sonnet 4.5, Haiku 4.5, Opus 4.5. Also refresh-on-read.
- **OpenRouter (Gemini path)**: 5-min TTL that does NOT update on read (fixed window) — gateway-specific behavior worth checking when going through proxies.

### Arbitrary / configurable TTL
- **Google Gemini explicit caching**: No minimum or maximum bounds on TTL. Default 60 min. You can `update` TTL on an existing cache and `delete` it early to stop billing. Billed as `cached_tokens × storage_duration` (per token-hour), not via a write-time premium.

### Opaque / provider-managed retention
- **OpenAI**: No exposed TTL. Baseline ~5–10 min of idle retention; off-peak can persist up to 1 hour. Extended prompt caching retains KV tensors 1–2h typical, up to 24h max.
- **DeepSeek, Grok, Moonshot, Groq, Kimi K2**: Automatic, provider-managed, no exposed TTL.

## Implicit vs explicit control

- **Implicit (zero-config)**: OpenAI, DeepSeek, Grok, Moonshot, Groq, Gemini implicit tier. Server decides what to cache when it detects a recurring prefix.
- **Explicit (marked / lifecycle-managed)**: Anthropic and Alibaba use inline `cache_control: {"type": "ephemeral"}` markers. Gemini explicit caching exposes full CRUD on cache objects via API (`create`, `get`, `update`, `delete`) — caches behave like first-class resources, similar to Valkey keys.

## Cache breakpoints / layering

Anthropic supports up to 4 `cache_control` breakpoints in a single request. You can mix TTLs within one request, but **longer TTL blocks must appear before shorter TTL blocks** in the prompt structure (tools → system → messages order). Practical use: 1-hour cache for stable system prompt + tool defs, 5-min cache for mid-conversation context, paying the higher write premium only on the truly stable prefix.

OpenAI caches in 128-token increments above a 1,024-token prefix floor. No user-controllable breakpoints.

## Pricing model dimensions

| Provider | Write cost | Read cost | Storage cost |
|---|---|---|---|
| Anthropic (5-min) | 1.25× base input | ~10% of base | none |
| Anthropic (1-hour) | 2× base input | ~10% of base | none |
| OpenAI | base input | ~50% of base | none |
| DeepSeek | base input | ~10% of base | none |
| Gemini explicit | base input | ~10% on 2.5+, ~25% on 2.0 | per cached-token-hour |
| Gemini implicit | base input | 10% on cache hit | none |

The storage-time model on Gemini explicit inverts the incentive: a long TTL on idle content costs money even if unused, whereas longer TTLs on Anthropic just mean a one-time higher write premium and want lots of reads to amortize.

## Cache scope / isolation

- **Anthropic**: Workspace-level isolation (moved from org-level on Feb 5, 2026). Cache entries do not cross workspaces inside the same org.
- **OpenAI**: Server-affinity routing — cache hits depend on landing on the same machine that processed the prior request. No explicit user control; affects perceived hit rate at scale.
- **Gemini**: Project-scoped.

## Client-side warming strategies

- **Keepalive pings**: Tiny request every TTL−1 min hitting the cached prefix to reset the sliding window. Cheap on Anthropic since pings are cache reads (~10% of base input).
- **Pre-warming on session start**: Fire one cheap write request before the user's first real interaction to hide the write latency.
- **Cache-aware request routing**: For server-affinity providers (OpenAI), sticky session IDs or consistent hashing on a proxy can improve hit rates.

## Self-hosted runtimes (vLLM, SGLang)

Full menu when running inference yourself:
- LRU / LFU eviction policies, or custom
- Prefix-tree (radix) cache vs flat KV cache
- Configurable memory budgets per node
- Pinning specific prefixes
- Cross-request KV reuse across multi-turn conversations
- Manual invalidation

Closest analog to managing your own Valkey cache layer.

## Adjacent: semantic caching (distinct from prompt/KV caching)

Prompt/KV caching is **exact-prefix match**. Semantic caching keys on embedding similarity of the query and caches *responses* — fundamentally different layer. Tools: GPTCache, Redis/Valkey + embeddings, Portkey, Helicone. Trade-offs: false-positive cache hits, embedding compute cost, but big wins when traffic has high semantic overlap without exact prefix overlap. Often layered on top of provider-side KV caching.

## Quick decision notes

- **Workload with stable system prompt, sporadic requests >5 min apart**: Anthropic 1-hour TTL or keepalive pings on 5-min.
- **Workload with very long static context, infrequent reuse over hours**: Gemini explicit caching with a custom TTL sized to expected reuse window.
- **High-volume API traffic with naturally repetitive prefixes**: OpenAI or DeepSeek — zero-config wins.
- **Heterogeneous prompts with semantic overlap but no exact prefix**: build a semantic cache layer.
- **Need deterministic eviction control**: self-host vLLM/SGLang.
