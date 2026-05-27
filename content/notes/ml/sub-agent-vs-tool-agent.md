---
title: Sub-Agent vs Tool-Agent in AI Harness Engineering
description: Distinguishes sub-agents from tool-agents by autonomy, interface, context, and harness contract.
tags:
  - ai-harness
  - agents
  - architecture
date: "2026-05-25T17:16:33.760075Z"
lastmod: "2026-05-25T22:47:08+05:30"
draft: false
---
# Sub-Agent vs Tool-Agent in AI Harness Engineering

A sub-agent is another agentic process delegated a goal. It has its own prompt/context, can reason over steps, may call tools, and returns a synthesized result or handoff. Use it when the work benefits from independent judgment.

Example: Investigate why the auth tests are flaky and report root cause plus fix options.

A tool-agent is a tool-shaped interface that may internally use agentic behavior, but from the harness perspective it is invoked like a tool: bounded input, bounded output, narrower contract. Use it when you want a capability, not an independent collaborator.

Example: Run code search and summarize where SessionManager is used, or open a browser, click through login, and report the screenshot result.

Practical distinction:

| Aspect | Sub-agent | Tool-agent |
|---|---|---|
| Harness role | Delegated worker | Callable capability |
| Autonomy | High | Usually constrained |
| Interface | Goal/task prompt | Tool schema or command-like call |
| Context | Often has its own working context | Usually receives explicit inputs |
| Output | Judgment, plan, result, handoff | Structured result, observation, action result |
| Best for | Parallel reasoning, review, implementation, investigation | Browser control, repo search, test running, retrieval, diagnostics |

Mental model: a sub-agent is someone you assign work to; a tool-agent is something you operate through.

The line can blur. A browser tool-agent might plan clicks internally, and a sub-agent might be wrapped as a callable tool. The engineering distinction is the contract: sub-agents are goal-directed collaborators; tool-agents are capability endpoints with tighter I/O and lifecycle boundaries.
