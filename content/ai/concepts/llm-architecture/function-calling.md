---
title: Function Calling Support in LLM Models
description: 'Explains function calling (tool use) in LLMs: how models emit structured requests to invoke external functions, the request-execute-return loop, provider support, and practical reliability notes.'
tags:
  - llm
  - function-calling
  - tool-use
  - mcp
  - agents
date: "2026-06-26T15:44:27Z"
lastmod: "2026-06-26T15:45:09Z"
draft: false
---

Function calling (also called "tool use") is the ability of an LLM to output a structured request to invoke an external function, rather than only producing free text. You give the model a set of function definitions — each with a name, description, and a JSON schema of parameters — and when the model decides a function is needed to answer, it returns a structured object naming the function and the argument values to pass. Your application code actually runs the function, then feeds the result back so the model can continue.

## How the loop works in practice

1. You send the user's message plus a list of available tools/functions (name + description + parameter schema).
2. The model either replies with text, or emits a "tool call" — JSON arguments matching the schema.
3. Your code executes the real function (API call, DB query, calculation, etc.).
4. You return the result to the model, which incorporates it into a final answer. This can repeat over multiple turns (and multiple parallel calls).

## Why it matters

It bridges the model to live data and real actions (weather, search, code execution, sending email), enforces structured output, and is the foundation of agents and the Model Context Protocol (MCP).

## Support across major providers

All major providers support it natively:

- **OpenAI** — "function calling"/tools, supports parallel calls and a strict JSON-schema mode.
- **Anthropic (Claude)** — "tool use," including parallel tools; underpins MCP.
- **Google Gemini** — function calling with parallel and compositional calls.
- **Meta Llama, Mistral, Cohere, and most recent open models** — support tool calling, though reliability varies by model size and fine-tuning.

## Practical notes

Smaller or older models are less reliable at choosing the right tool and producing valid arguments; quality depends heavily on clear function descriptions and schemas; and the model only *requests* a call — your code is responsible for executing it and handling errors or hallucinated arguments.