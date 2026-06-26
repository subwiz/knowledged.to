---
title: Local + Frontier Model Collaboration Patterns in Open Source Harnesses
description: As local LLMs improve, harnesses are learning to pair them with frontier models. A look at the four collaboration patterns already shipping in open source.
tags:
  - llm
  - agents
  - harness
  - local-models
  - routing
  - speculative-decoding
date: "2026-06-24T17:08:01Z"
lastmod: "2026-06-26T16:26:12Z"
draft: false
---

As local LLM capabilities improve, harness architecture is evolving to let local models collaborate with frontier models. The patterns already exist in open source, clustering into four distinct architectures that solve different problems.

## 1. Routing / cascades (most mature, most common)

One-model-per-query selection: a cheap classifier inspects the prompt and sends easy queries to a small/local model, hard ones to a frontier model.

- [RouteLLM](https://github.com/lm-sys/RouteLLM) (LMSYS) — canonical implementation. Trains routers (BERT, matrix factorization, causal-LLM) on preference data; explicitly supports a [local model as the weak endpoint](https://github.com/RouteLLM/blob/main/examples/routing_to_local_models.md). Reports ~95% of GPT-4 quality at ~26% of the calls.
- [LLMRouter](https://github.com/ulab-uiuc/LLMRouter), [anyscale/llm-router](https://github.com/anyscale/llm-router) — similar.

Collaboration only in the weak sense: models don't interact, they're alternatives.

## 2. Advisor / worker-calls-expert (genuinely collaborative)

A local worker model runs the task and calls up to a frontier model as a tool when stuck. Cleanest public example: Harvey, where an open-source worker with a callable frontier "advisor" beat a frontier model on both quality and cost. Closest to true co-execution within a single task, but still relatively rare in open harness code compared to routing.

## 3. Speculative decoding (collaboration at token level)

llama.cpp, vLLM, SGLang, and LM Studio ship draft-model speculative decoding: a small local model proposes tokens, the larger model verifies them in a batch.

Key caveat: this cannot cross the local-to-remote-API boundary, because verification needs the big model's raw logits, which hosted frontier APIs don't expose. So it stays co-located (small + large, both local). Also breaks down on MoE models like Gemma 4, where verification forces loading the union of activated experts.

## 4. Harness-level model tiering

Claude Code dispatches different tiers (Opus/Sonnet/Haiku) per subtask, and exposes `ANTHROPIC_BASE_URL` plus `CLAUDE_CODE_SUBAGENT_MODEL` to point the whole thing — or just subagents — at a local server (Ollama, LM Studio, llama.cpp with native Anthropic Messages API support). [little-coder](https://github.com/itayinbarr/little-coder) is a harness explicitly optimized for smaller models. This is the infrastructure layer that makes heterogeneous local+frontier collaboration configurable.

## Summary

Well-represented in OSS today: cost-based routing (pick one) and co-located speculative decoding (token-level). The emerging vision — a local model as the persistent driver that delegates hard sub-reasoning to a frontier model mid-task — exists (advisor pattern, plus research like NVIDIA's "Small Language Models are the Future of Agentic AI," which found SLMs can handle ~60% of MetaGPT's agent queries) but is the least standardized. The harness primitives to build it (subagent model overrides, tool-callable model endpoints, base-URL redirection) are already in place; the orchestration policies are where open source is still thin.

## Sources

- https://github.com/lm-sys/RouteLLM
- https://github.com/RouteLLM/blob/main/examples/routing_to_local_models.md
- https://github.com/ulab-uiuc/LLMRouter
- https://github.com/anyscale/llm-router
- https://github.com/itayinbarr/little-coder
- https://medium.com/@luongnv89/how-to-run-claude-code-codex-with-local-models-via-llamacpp-ollama-lmstudio-and-vllm-2026-7d00ba7e63a4
- https://www.kdnuggets.com/pairing-claude-code-with-local-models
- https://www.mindstudio.ai/blog/hybrid-ai-architecture-local-models-cloud-frontier