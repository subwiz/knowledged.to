---
title: 'RLVR vs. the Agent Loop: Training-Time vs. Inference-Time'
description: Distinguishes RLVR as training-time weight updates from inference-time agent verification loops.
tags:
  - llm
  - rl
  - rlvr
  - agents
  - inference
  - training
date: "2026-06-24T06:21:24Z"
lastmod: "2026-06-24T06:21:43Z"
draft: false
---

Coding agents like Codex and Claude Code visibly verify their output against a hypothesis and iterate — propose, run a tool, read the result, correct, proceed. This looks like RLVR (RL with verifiable rewards), but it is **not** reinforcement learning. The two are easy to confuse because they share an ingredient, but they operate at fundamentally different levels.

## The defining line: do the weights change?

The defining feature of reinforcement learning is that a reward signal is used to **update the model's weights** via gradient descent. Learning, in the technical sense, means the parameters change.

**RLVR is a training-time procedure.** You sample many trajectories, score them with an automatic verifier (tests pass, math answer correct), compute a gradient that makes high-reward trajectories more likely, and permanently bake that into the weights. After training, the model is frozen.

**The agent loop is inference-time.** The weights are completely static — nothing is being learned. The agent forms a hypothesis, runs a tool, reads the result, and conditions its next token prediction on the new context. The "verify → proceed" cycle is just in-context reasoning over a growing transcript. No reward is computed, no gradient flows, nothing persists after the session ends. Close the session and the model is exactly as it was.

## Why they look identical

RLVR during training is essentially teaching the model the *policy* of doing exactly this loop — propose, test, observe, correct. The agentic behavior you watch at inference is the **learned policy being executed**, not the learning itself.

It's the difference between a chess player who trained by playing thousands of games (the RL) and that same player thinking through a single game move-by-move (the inference loop). The deliberation in one live game isn't "learning chess," even though it's the visible product of having learned.

Another analogy: training is to inference as evolution is to an organism's lifetime behavior. Evolution (RL) shapes instincts over generations by selection pressure; an animal foraging and adjusting to feedback within its lifetime (the agent loop) uses those instincts rather than evolving them.

## Two clarifications that trip people up

**The verifier is the genuinely shared ingredient.** The same test suite or checker can serve as a *reward function* during RLVR and as a *tool the agent calls* at inference. Same signal, totally different use — one updates weights, the other just enters the context window.

**"Not learning in-session" doesn't mean nothing is retained.** The context window acts as temporary scratch memory, and across sessions things like fine-tuning on logged trajectories or memory files can fold experience back in. But those are separate, deliberate training steps — not the live loop.

## Bottom line

The visible verify-and-proceed behavior is the agent loop working. RLVR is what made the model good at running that loop; the loop you observe is the payoff, not the training.