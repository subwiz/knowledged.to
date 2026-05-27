---
title: LLM Thinking Token Budgets
description: Explains thinking-token budget parameters, provider naming, cost-latency tradeoffs, and completion-cap interactions.
tags:
  - llm
  - api
  - tokens
  - reasoning
date: "2026-05-25T17:09:36.967114Z"
lastmod: "2026-05-25T22:48:50+05:30"
draft: false
---
# Thinking Token Budget

Token budget parameters for thinking LLMs usually cap how many internal reasoning tokens the model may spend before producing the visible answer.

Common names by API/provider include:
- max_tokens / max_output_tokens: caps generated output tokens, sometimes including hidden reasoning tokens depending on the API.
- reasoning_effort: qualitative budget like low, medium, high; the API maps this to an internal reasoning-token allowance.
- thinking_budget / budget_tokens: explicit number of hidden reasoning tokens allowed for models that expose thinking controls.
- max_completion_tokens: in some APIs, caps both reasoning tokens and final answer tokens together.

Why it matters:
- Higher budget: useful for hard math, coding, planning, and multi-step debugging.
- Lower budget: cheaper, faster, enough for simple Q&A or formatting tasks.
- Too low: model may answer prematurely or miss steps.
- Too high: slower and more expensive, sometimes overthinks simple tasks.

Mental model:
total completion budget = hidden reasoning tokens + visible answer tokens

If the completion cap is tight, a thinking model may spend tokens reasoning and have too little room left for the final answer.

Example qualitative setting:
{
  "model": "reasoning-model",
  "reasoning_effort": "medium",
  "max_output_tokens": 1000
}

Example explicit thinking budget:
{
  "thinking": {
    "type": "enabled",
    "budget_tokens": 2048
  },
  "max_output_tokens": 1000
}

The exact parameter name depends on the model provider and API.