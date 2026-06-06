---
title: Agent Harness Six Components (E, T, C, S, L, V) — Survey Summary
description: 'Summarizes the six-component agent harness model: execution, tools, context, state, hooks, and evaluation.'
tags:
  - ai-agents
  - evaluation
  - llm
  - harness
  - architecture
date: "2026-06-05T18:19:54Z"
lastmod: "2026-06-05T18:44:58Z"
draft: false
---

The [Agent Harness for LLM Agents survey](https://www.preprints.org/manuscript/202604.0428) (April 2026, 110+ papers, 23 systems) formalizes the agent harness as a tuple **H = (E, T, C, S, L, V)** — six runtime governance functions sitting between the model and the external world.

## The six components

**E — Execution loop.** Manages the observe-think-act cycle: turn sequencing, termination conditions, error recovery. Formalized as a labeled transition system (LTS) with states idle → invoking-model → dispatching-tool → awaiting-tool-result → committing-state → terminated. The formalism yields three checkable properties: safety (termination always reachable — no execution runaway), liveness, and determinism (the transition function must be a function, with environment nondeterminism isolated at tool-call boundaries). Failure mode if missing: execution runaway.

**T — Tool registry.** A typed, validated catalog of tool interfaces; routes and monitors invocations. Failure mode: tool misuse.

**C — Context manager.** Governs what enters the model's context window each turn — compaction, retrieval, prioritization. Failure mode: context blowout.

**S — State store.** Persists task state across turns (optionally sessions) and enables recovery from partial failures. Failure mode: state loss under crash.

**L — Lifecycle hooks.** Pre-/post-invocation interception points for auth, audit logging, policy enforcement, instrumentation. Failure mode: unmonitored side effects.

**V — Evaluation interface.** Distinct from L even though both "record activity": L tells you *that* a tool call happened (operational logging); V emits trajectories in a canonical schema — action sequences with typed arguments, intermediate state snapshots, tool-call outcomes, goal-completion signals, per-step token usage — consumable directly by benchmarking pipelines like HAL. A harness with only operational logging cannot support standardized cross-model evaluation. Failure mode: unobservable behavior.

## Structural rules

E + T are the necessary minimum to count as a harness at all (otherwise it's an inference wrapper or a reasoning engine with no effectors). All six with production-grade reliability — error handling, auth, observability integrations, documented failure modes — makes a "full-stack harness."

## System classification via LTS analysis

- **ReAct**: primitive non-harness — collapsed two-state LTS, no error-recovery arc, partial transition function (safety fails).
- **AutoGPT**: monolithic harness — total transition function with explicit error recovery, but weak implementation guarantees (no idempotent state writes or atomic commits).
- **LangGraph**: topology-encoded harness — liveness guaranteed by DAG acyclicity, but the C-component exists only implicitly in graph structure rather than as an active runtime policy.