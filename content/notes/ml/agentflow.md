---
title: AgentFlow
description: Overview of AgentFlow, an agent architecture that trains a planner with Flow-GRPO for multi-turn tool use.
tags:
  - ai
  - llm
  - agents
  - reinforcement-learning
  - tool-calling
  - agentic-ai
  - research
  - iclr-2026
  - multi-turn
  - planning
date: "2026-05-19T05:08:59.082315Z"
lastmod: "2026-05-19T10:39:29+05:30"
draft: false
---
# AgentFlow: In-the-Flow Agentic System Optimization

**Source:** arXiv:2510.05592 — ICLR 2026 Oral (Top 1.1%)  
**Authors:** Zhuofeng Li, Haoxiang Zhang, Seungju Han, Sheng Liu, Jianwen Xie, Yu Zhang, Yejin Choi, James Zou, Pan Lu (Stanford University, Texas A&M, UC San Diego, Lambda)

## The Problem It Solves

Standard tool-augmented LLMs (like Search-R1 or ToRL) train a single monolithic policy that interleaves thinking and tool calls in one big context. This works okay on short tasks but scales poorly on long-horizon problems: the context grows, the reward signal is sparse (you only find out at the very end whether you succeeded), and the model generalizes weakly to new tool configurations. AgentFlow is built to fix all three of those.

## The System: Four Specialized Modules

AgentFlow decomposes the agent into four roles connected through a shared evolving memory:

- **Planner** — the only trainable module; a policy (πθ) that looks at the query, the available tools, and the current memory state, then decides what to do next: which sub-goal to pursue and which tool to call.
- **Executor** — actually runs the tool and returns results.
- **Verifier** — checks whether the result solves the sub-goal, producing a binary yes/no signal. If no, memory is updated and the planner tries again.
- **Generator** — when the verifier says yes (or the turn budget is exhausted), takes the full memory and produces the final answer.

The key design choice: only the Planner is trained. The other three modules can be anything (frozen LLMs, rule-based systems, external APIs), and the system still benefits from training the planner on-policy in the live multi-turn environment.

## The Training Algorithm: Flow-GRPO

This is the paper's main technical contribution. The challenge is that RL across multi-turn trajectories is hard: credit assignment is tricky (which of the 10 turns was responsible for success or failure?), and the full trajectory is too long to optimize in one shot.

Flow-GRPO solves this with two ideas:

**1. Broadcast a single trajectory-level reward to every turn.** Rather than trying to assign partial credit to each step, every action in the trajectory gets the same reward — 1 if the final answer was correct, 0 if not (evaluated by an LLM-as-judge). If the overall trajectory succeeded, every decision along the way is reinforced.

**2. Group-normalize advantages across parallel rollouts.** For each query, the system samples G trajectories in parallel. The advantage for each trajectory is normalized by the group mean and standard deviation — the same idea as GRPO — keeping training stable even with sparse rewards.

The combination turns intractable multi-turn RL into a sequence of tractable single-turn policy updates.

**Critical finding:** Offline SFT as a baseline caused a catastrophic **19% performance collapse**. Online RL (Flow-GRPO) gave a **17.2% improvement**. The on-policy, in-the-flow nature of training is essential — you can't learn from static demonstrations.

## Results

Tested across 10 benchmarks with a 7B backbone (Qwen-2.5-7B), outperforming GPT-4o:

| Task Type | Benchmarks | Gain over baselines |
|-----------|-----------|---------------------|
| Search | Bamboogle, 2Wiki, HotpotQA, Musique | +14.9% |
| Agentic | GAIA | +14.0% |
| Math | AIME 2024, AMC 23, Game of 24 | +14.5% |
| Scientific | GPQA, MedQA | +4.1% |

**Additional scaling findings:**
- Performance keeps improving as inference turns increase from 3 to 10
- Consistent gains across backbone sizes from 3B to 7B
- If internal tool engines are upgraded (e.g. 7B → GPT-4o tools), performance improves further without retraining

## Practical Relevance for AI Engineers

The model and code are open source. If you're building a multi-step agent — anything that calls tools across multiple turns — the AgentFlow architecture is a concrete blueprint:
- Separate planner, executor, verifier, and generator
- Train only the planner, on-policy, with trajectory-level rewards
- Use Flow-GRPO for stable multi-turn RL

Code: https://github.com/lupantech/AgentFlow  
Model: https://huggingface.co/AgentFlow  
Demo: https://huggingface.co/spaces/AgentFlow/agentflow

## References
- Project page: https://agentflow.stanford.edu/
- Paper: https://arxiv.org/abs/2510.05592
- GitHub: https://github.com/lupantech/AgentFlow