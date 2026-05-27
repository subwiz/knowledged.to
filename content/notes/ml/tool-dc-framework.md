---
title: Tool-DC Framework
description: Overview of Tool-DC, a try-check-retry framework for robust long-context tool-calling with large tool registries.
tags:
  - ai
  - llm
  - tool-calling
  - agents
  - research
  - agentic-ai
  - function-calling
date: "2026-05-19T03:53:48.391602Z"
lastmod: "2026-05-19T09:24:16+05:30"
draft: false
---
# Tool-DC Framework: Try, Check and Retry for Long-context Tool-Calling

**Source:** arXiv:2603.11495 — Accepted at ACL 2026  
**Authors:** Kunfeng Chen, Qihuang Zhong, Juhua Liu, Bo Du (Wuhan University), Dacheng Tao (NTU)

## The Core Problem

When you give an LLM access to a large library of tools — say 20, 50, or hundreds of APIs — performance degrades sharply. The paper shows that even going from fewer than 10 tools to 20 causes significant accuracy drops across all tested models, especially smaller ones. Two things go wrong: the sheer length of the context buries the signal, and semantically similar tools with slightly different argument schemas confuse the model when it's trying to fill in the right parameters.

## The Try-Check-Retry Pipeline (Training-free variant)

Tool-DC's training-free version (TF) is a divide-and-conquer inference wrapper you can drop onto any model without retraining.

**Try — Grouping and Local Inference.** Rather than showing the model all N tools at once, Tool-DC first uses a retriever (e.g. BM25) to pull the top-K most relevant tools. It then constructs K parallel groups: each group has one of those top-K tools as an "anchor," plus a disjoint subset of lower-ranked tools from the remainder. The key insight is that each anchor tool gets its own group, which prevents similar tools from competing with each other in the same context. The model then runs local inference independently on each group, outputting either a tool call (tool name + arguments) or a null token.

**Check — Schema Consistency Validation.** Each local output is filtered by a rule-based validator against three constraints: the function name must exist in the tool set, all required argument keys must be present, and argument values must match the defined data types. This step catches hallucinated function names and malformed argument structures before they propagate. The valid outputs form a refined candidate set.

**Retry — Global Aggregation.** The original tool definitions for everything in the validated candidate set are retrieved and assembled into a much smaller, high-signal context. The model then makes a final global call over this clean subset — essentially getting a second pass where the noise has been filtered out and it can self-refine.

## The Training-based Variant (TB)

The TF version requires multiple forward passes, which adds latency. The training-based version (TB) addresses this by internalizing the Try-Check-Retry reasoning into the model weights via fine-tuning. The process: run TF on a training dataset, collect the successful reasoning traces (local inference → validation → global decision), synthesize those into Chain-of-Thought data with a structured rationale template (Candidate Selection → Validation → Final Review), and fine-tune the model on it. At inference time, the model executes the same reasoning in a single forward pass.

**Result:** Using TB, Qwen2.5-7B scores 83.16% on the Berkeley Function-Calling Leaderboard, surpassing OpenAI o3 and Claude Haiku 4.5.

## Results

- Tool-DC (TF): up to **+25.10% average gains** on BFCL and ACEBench benchmarks vs. baseline
- Tool-DC (TB): Qwen2.5-7B reaches **83.16% on BFCL**, outperforming proprietary models including OpenAI o3 and Claude Haiku 4.5

## Practical Relevance for AI Engineers

If you're building agents with large tool registries — MCP servers, API-heavy workflows, or any system where the model chooses from dozens of functions — Tool-DC is directly applicable. The TF variant is plug-and-play with no training required. The TB variant is worth exploring if you're fine-tuning a smaller open model and want to close the gap with proprietary models on tool-calling tasks.

## References
- Paper: https://arxiv.org/abs/2603.11495
- Full HTML: https://arxiv.org/html/2603.11495