---
title: Tempo Framework
description: Overview of Tempo, a query-aware temporal compression framework for long-video understanding in multimodal AI, using a small VLM to filter relevant frames before passing a condensed representation to a large model
tags:
  - notes
date: "2026-04-16T22:15:24+05:30"
lastmod: "2026-04-16T22:15:24+05:30"
draft: false
---

Tempo is a framework designed to solve one of the hardest problems in multimodal AI: **understanding very long videos** without blowing up your context window or compute budget.

## The Core Problem It Solves

Videos are brutally expensive for transformers. A 1-hour video at even 1 frame per second gives you 3,600 frames. At typical vision encoding resolutions, each frame becomes hundreds of tokens — potentially millions of tokens total, far beyond what any current model can process in a single context window. And even if it could, the attention computation would be prohibitively expensive (attention is O(n²) in sequence length).

Prior approaches mostly resorted to uniform frame sampling — just pick every Nth frame and hope you don't miss anything important. This works poorly for real-world videos where interesting events are sparse and unevenly distributed.

## Tempo's Approach

Tempo introduces a **query-aware temporal compressor** built around a Small Vision-Language Model (SVLM). The key word is *query-aware* — rather than compressing the video uniformly, Tempo compresses it differently depending on what question you're actually asking.

### Pipeline

1. **Frame encoding** — video frames are encoded into visual embeddings as usual.
2. **Query-aware compression** — the SVLM takes both the visual embeddings *and* the user's query, then identifies which temporal segments are relevant to that specific question. Relevant frames/segments are preserved at higher fidelity; unrelated stretches get aggressively compressed or dropped.
3. **Compressed representation passed to the main MLLM** — the large multimodal model receives a much shorter, query-focused token sequence rather than the raw full video, and generates the answer.

## Why the SVLM Approach Is Smart

Using a small model for compression is elegant:
- The SVLM is cheap to run — its job isn't to *answer* the question, just to *identify relevance*.
- It acts as a smart pre-filter, doing temporal attention at a coarse level so the expensive large model only reasons over the parts that matter.
- Architecturally similar to RAG: a retriever (cheap) narrows down context before the generator (expensive) does the heavy lifting — just applied to the time dimension of video rather than a document corpus.

## What It Enables

- **Hour-long video Q&A** — e.g., "what was the presenter doing when they mentioned the revenue figures?" over a full lecture or meeting recording
- **Video summarization with focus** — "summarize only the parts relevant to the product demo"
- **Temporal grounding** — "when does X happen?" over long content
- **Surveillance and monitoring** — finding specific events in hours of footage without manual scrubbing

## Limitations and Open Questions

- Compression quality depends on the SVLM's ability to judge relevance — made before the large model sees full context.
- If the query is ambiguous, the SVLM might compress away important material.
- Queries requiring understanding of *patterns across time* (e.g., "how does the speaker's tone change over the talk?") are harder than point-in-time retrieval.

## Broader Significance

Tempo is part of a broader wave of research on **hierarchical / cascaded multimodal processing** — the principle that raw, unfiltered perceptual data should not go directly to your most expensive model. Use cheap, fast models for coarse filtering and structuring, then hand off a condensed, task-relevant representation to the powerful model. This pattern is likely to become standard practice as video becomes a primary modality in AI applications.