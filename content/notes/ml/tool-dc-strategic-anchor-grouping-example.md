---
title: Tool-DC Strategic Anchor Grouping — Web Search Example
description: Concrete web-search example showing how Tool-DC strategic anchor grouping reduces schema-confusion in tool calls.
tags:
  - ai
  - llm
  - tool-calling
  - agents
  - tool-dc
  - research
  - agentic-ai
  - function-calling
  - example
date: "2026-05-19T06:12:48.497082Z"
lastmod: "2026-05-19T11:43:24+05:30"
draft: false
---

This is a concrete example illustrating how the Strategic Anchor Grouping mechanism works in the Tool-DC framework. See also: `notes/ml/tool-dc-framework.md`.

## Setup

Query: **"search the web for recent AI news"**  
Tool library: 20 tools total  
Retriever returns top 3:

- **T_top** = [Google Search, Bing Search, DuckDuckGo Search]
- **T_tail** = 17 remaining tools (Calculator, Weather API, Wikipedia, Code Executor, etc.)

With K=3, Tool-DC creates **4 groups**:

---

## S₀ — Full top-K group (kept as baseline)

```
[Google Search, Bing Search, DuckDuckGo Search]
```

This is the problematic group. All three tools do essentially the same thing — search the web — but have slightly different argument schemas:
- Google Search: `query` + `num_results` + `safe_search`
- Bing Search: `query` + `count` + `market`
- DuckDuckGo Search: `query` + `region`

The model sees three nearly-identical tools and gets confused: it might call `Bing` but fill in `num_results` (Google's argument), producing a hallucinated, schema-invalid call.

---

## S₁ — Google Search as anchor

```
[Google Search, Calculator, Weather API, Code Executor, Wikipedia]
```

Google Search is the only web search tool in the group. The distractors are clearly unrelated. The model has no trouble picking Google Search and correctly filling `query`, `num_results`, and `safe_search` because there's nothing competing for its attention.

---

## S₂ — Bing Search as anchor

```
[Bing Search, Database Query, Email Sender, File Reader, Translator]
```

Bing is the clear winner in this group. The model correctly calls Bing with `query`, `count`, and `market` — no confusion with Google's schema.

---

## S₃ — DuckDuckGo Search as anchor

```
[DuckDuckGo Search, Image Generator, Slack Messenger, Calendar API, Currency Converter]
```

DuckDuckGo stands out cleanly. Arguments filled correctly.

---

## After Parallel Inference: Check Step

The Check step validates each output against its schema:

- S₀ → ❌ invalid (model called `Bing` but used `num_results` — wrong argument key)
- S₁ → ✅ `Google Search(query="recent AI news", num_results=5, safe_search=true)`
- S₂ → ✅ `Bing Search(query="recent AI news", count=5, market="en-US")`
- S₃ → ✅ `DuckDuckGo Search(query="recent AI news", region="us-en")`

---

## Retry Step

Three valid candidates. The Retry step assembles a clean group from just those three tools and gives the model one final pass:

```
[Google Search, Bing Search, DuckDuckGo Search]
+ prior validated outputs as context
```

This looks like S₀ again — but now the model has seen the validated, schema-correct calls for each tool as context. Instead of starting cold trying to pick between three identical-seeming tools, it's refining a decision informed by what already passed validation. It might pick Google Search as the final answer, now with the correct arguments.

---

## Core Intuition

The anchor groups aren't about finding the *best* tool in isolation — they're about giving each similar tool a fair, low-noise environment to prove it can produce a valid call. The confusion between Google/Bing/DuckDuckGo doesn't happen because they're never in the same context during the Try phase. The Retry phase then handles the final disambiguation, but with schema-validated evidence in hand rather than starting from scratch.

**Key points:**
- Similar tools are deliberately **separated** into different anchor groups
- Each anchor is paired with clearly unrelated distractors from T_tail
- Distractors are disjoint across groups (no tool appears twice)
- T_tail tools get coverage — the correct tool might have been ranked outside top-K by the retriever
- The Retry step is not starting fresh — it has schema-validated evidence to work from