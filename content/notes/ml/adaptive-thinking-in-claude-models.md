---
title: Adaptive Thinking in Claude Models
description: Explains Claude adaptive thinking, effort levels, fixed-budget deprecation, and hidden reasoning display.
tags:
  - ai
  - claude
  - llm
  - reasoning
  - anthropic-api
date: "2026-06-12T06:50:25Z"
lastmod: "2026-06-12T06:51:10Z"
draft: false
---

Adaptive thinking is a feature of recent Claude models (Opus 4.6 and later, Sonnet 4.6, Fable 5) where the model itself decides **when** to engage in extended reasoning and **how much** to think, on a per-request basis — rather than the developer allocating a fixed reasoning budget up front.

## The problem it solves

Earlier "extended thinking" required a fixed token budget (`budget_tokens`) for the model's internal reasoning. That was awkward: a simple question wasted the budget, a hard problem might need more, and the number had to be tuned per workload. Adaptive thinking replaces that — the model looks at the task and dynamically scales its reasoning depth, including interleaving thinking between tool calls in agentic workflows (which previously needed a separate beta flag).

## Usage (Claude API)

```python
response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},  # optional: low | medium | high | xhigh | max
    messages=[{"role": "user", "content": "Solve this step by step..."}],
)
```

## Key points

- **Pairs with the `effort` parameter**, the remaining control knob: instead of a token count, you set a qualitative level (`low` through `max`) that influences thinking depth, tool-call consolidation, and verbosity. `high`/`xhigh` suit coding and agentic work; `low` suits simple, latency-sensitive tasks.
- **Fixed budgets are gone on the newest models.** On Opus 4.7/4.8 and Fable 5, sending `budget_tokens` returns a 400 error; on Opus 4.6 and Sonnet 4.6 it still works but is deprecated. Adaptive is the only on-mode going forward.
- **Not on by default** — omitting the `thinking` parameter means no thinking. Opt in explicitly with `{"type": "adaptive"}`.
- **Thinking text is hidden by default** on Opus 4.7+ — reasoning blocks stream but are empty unless you set `display: "summarized"`, useful for showing users reasoning progress.

## Broader context

The general idea — sometimes called dynamic or conditional compute in the research literature — is that reasoning effort should match problem difficulty: spend little compute on trivial questions and a lot on multi-step problems, with the model making that allocation itself rather than the developer guessing.
