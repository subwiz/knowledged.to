---
title: CompactRAG
description: Explains CompactRAG, a multi-hop RAG method using offline atomic QA pairs and fixed two-call inference.
tags:
  - ai
  - RAG
  - multi-hop
  - LLM
  - retrieval
  - efficiency
date: "2026-05-29T16:30:24Z"
lastmod: "2026-05-29T16:49:05Z"
draft: false
---

# CompactRAG

*Research summary — May 29, 2026. Source: arXiv:2602.05728 (appearing at WWW '26)*

---

## The Problem It Solves

Standard iterative RAG for multi-hop questions (e.g., "Who directed the film starring the actor born in the same city as Marie Curie?") works by alternating between retrieval and LLM reasoning at each hop. This means:

- **N hops = N LLM calls** for reasoning, plus retrieval at each step
- High token overhead — each call re-reads growing context
- **Entity drift** — the entity being tracked across hops can get corrupted or lost as queries are reformulated each step

---

## The Core Idea: Decouple Offline and Online

CompactRAG does most of the work *once, offline*, rather than at query time.

**Offline stage** (done once per corpus):
An LLM reads the corpus and converts it into an **atomic QA knowledge base** — a collection of minimal, fine-grained question-answer pairs. Instead of storing raw passages, you store things like:
- Q: "Where was Marie Curie born?" → A: "Warsaw"

These atomic pairs are more semantically aligned with how questions get asked, reduce redundancy vs. raw chunking, and pre-resolve much retrieval ambiguity.

**Online stage** (at query time):
1. Decompose the complex query into sub-questions, with careful entity rewriting to preserve consistency across hops
2. **Dense retrieval** against the atomic QA knowledge base (not the original corpus)
3. **RoBERTa-based answer extraction** (not an LLM) to pull answers from retrieved pairs
4. One final LLM call to synthesize the full answer

**Total LLM calls at inference: 2**, regardless of hop count — one for decomposition, one for synthesis.

---

## Results

Benchmarked on **HotpotQA**, **2WikiMultiHopQA**, and **MuSiQue**. Achieves competitive accuracy against iterative RAG baselines while substantially reducing token consumption and LLM call count.

---

## Engineering Tradeoffs

- **Upfront cost**: Offline corpus conversion with an LLM (pay once per corpus)
- **Query-time savings**: Fixed 2 LLM calls regardless of hop depth — good deal at high query volume
- **Best for**: Relatively stable corpora with frequent multi-hop queries
- **Retrieval quality**: QA pairs are query-shaped, so dense retrieval tends to surface more precise matches than standard chunking

Worth benchmarking against standard chunking if you're seeing entity drift or high token costs in iterative RAG pipelines.

---

## Resources

- [Paper: arXiv:2602.05728](https://arxiv.org/abs/2602.05728)
- [HTML version](https://arxiv.org/html/2602.05728)
