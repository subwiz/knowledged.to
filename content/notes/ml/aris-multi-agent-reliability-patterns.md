---
title: 'ARIS: Multi-Agent Reliability Patterns'
description: Patterns from ARIS for reliable multi-agent research using adversarial review, audits, and persistent memory.
tags:
  - ai
  - multi-agent
  - reliability
  - aris
  - llm
  - agent-patterns
date: "2026-06-01T03:47:41Z"
lastmod: "2026-06-01T03:48:23Z"
draft: false
---

Source: arXiv:2605.03042 — ARIS: Autonomous Research via Adversarial Multi-Agent Collaboration (Shanghai Jiao Tong University, May 2026)

## Core Premise

> **Any long-term task performed by a single agent is unreliable.**

The failure mode ARIS solves isn't "the agent crashes." It's *plausible unsupported success* — an agent produces a confident, coherent output that is subtly wrong, hallucinated, or unsupported by actual evidence.

---

## Three Architectural Layers

**1. Execution Layer** — 65+ modular "skills" encoded as plain Markdown files (SKILL.md). Each skill has: inputs, outputs, step-by-step procedures, quality gates, and failure-handling instructions. Skills exchange state via versioned plain-text artifact files (not in-memory or DB), enabling checkpoint-based recovery across sessions.

**2. Orchestration Layer** — chains skills into end-to-end workflows. Workflows are resumable at any intermediate artifact (not monolithic scripts).

**3. Assurance Layer** — makes outputs trustworthy rather than just plausible. Structurally independent from the execution layer.

---

## Key Pattern: Cross-Model Adversarial Collaboration

A **critique-to-action loop** between two agents from *different model families*:

- **Executor** (e.g., Claude) drives forward progress
- **Reviewer** (e.g., GPT-5.4) critiques intermediate artifacts independently

Why different families? Same-model self-refinement loops share inductive biases — the generator and validator make correlated errors that neither catches. Cross-family critique produces genuinely divergent feedback.

**Critical protocol:** The reviewer reads artifacts *directly*, not via an executor-provided summary. If the executor summarizes first, the reviewer critiques the executor's framing, not the underlying work.

Loop terminates when: score > 6/10 AND all critical items resolved, OR max 4 rounds reached.

---

## The Assurance Stack

### Evidence-to-Claim Audit Cascade (3 stages)

**Stage 1: Experiment Integrity Audit**
A cross-model reviewer checks for 5 failure modes:
1. Model-derived reference labels (outputs used as ground truth)
2. Self-normalized scores (metrics that inflate by design)
3. Phantom results (claimed numbers not in actual output files)
4. Dead-code inflation (metrics defined but never executed)
5. Scope inflation (claims that generalize beyond what was tested)

**Stage 2: Result-to-Claim Mapping**
Each claim gets a verdict: *supported*, *partially supported*, or *invalidated*. Claims with Stage 1 failures cannot be marked fully supported.

**Stage 3: Paper-Claim Audit**
A *fresh, zero-context* reviewer (new thread, no history) cross-checks quantitative claims against raw evidence. Fresh-thread design prevents accumulated context from biasing the audit.

### Manuscript Assurance (additional checks)
- Five-pass scientific editing pipeline (clutter, active voice, structure, terminology consistency, numerical consistency)
- Proof verification with 20-category issue taxonomy
- Visual PDF review (catches layout issues source-only review misses)
- Citation audit: existence, metadata correctness, and context appropriateness

---

## Effort Presets

| Level | Multiplier | Use |
|---|---|---|
| lite | ~0.4x | Quick exploration |
| balanced | 1x | Default |
| max | ~2.5x | Thorough review |
| beast | ~5–8x | Maximum depth |

Reviewer reasoning effort (GPT-5.4 xhigh) stays constant regardless of preset — effort scaling changes coverage/iteration counts, not reviewer quality.

---

## Persistent Memory: Research Wiki

Four entity types stored as structured Markdown: papers, ideas, experiments, claims.
Eight typed relationships: extends, contradicts, addresses_gap, inspired_by, tested_by, supports, invalidates, supersedes.

Key design: **rejected ideas are retained** as a banlist. Without persistent memory, ideation pipelines re-propose the same dead-end directions across sessions.

All state lives in versioned text files on disk — not in LLM context windows — enabling cross-session continuity and checkpoint recovery.

---

## Actionable Patterns for Production AI Apps

1. **Use heterogeneous reviewers.** Route GPT → Claude reviewer or Claude → GPT reviewer. Correlated blind spots are real.
2. **Reviewer reads artifacts directly.** Don't pass the executor's summary — give the reviewer the raw output.
3. **Fresh context for audits.** Use a new thread/context for critical verification. Prior conversation history creates confirmation bias.
4. **Track claims explicitly.** Maintain a claim ledger mapping every factual/quantitative output to its supporting evidence. Don't trust the final prose.
5. **Persistent memory over ephemeral context.** State in versioned files enables checkpoint recovery and cross-session continuity.
6. **Cap review rounds.** Over-iterating with the same reviewer causes the executor to overfit to reviewer preferences rather than actual quality improvement.

---

## Limitations

- Assurance stack is advisory, not formal verification
- Repository-level review sends code to external APIs (confidentiality risk)
- Cross-family vs. same-family review hasn't been rigorously benchmarked yet (listed as future work)
- Human responsibility remains: ARIS automates execution and review loops; humans provide research direction and make final submission decisions

---

## Links

- Paper: https://arxiv.org/abs/2605.03042
- Code: https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep
