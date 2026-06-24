---
title: Coding LLM Training with SFT and Verifiable RL
description: Explains scripted coding-LLM training with teacher traces, synthetic bugs, tests, SFT, and verifiable RL.
tags:
  - llm-training
  - coding-agents
  - reinforcement-learning
  - supervised-finetuning
date: "2026-06-23T17:33:31Z"
lastmod: "2026-06-23T17:53:29Z"
draft: false
---

Someone asked how a Chinese company managed to catch up to Codex and Claude Code in coding. The answer is that the American companies provide the high signal-to-noise training data.

The way it works is as follows (all is scripted, no human in the loop):

1. You take a large enough base model and finetune it using a **combination of reinforcement learning and supervised finetuning**.
2. To get training examples, you ask some LLM to introduce a subtle bug into an existing codebase and provide a test script that would only return True when the bug is fixed.
3. You use Codex/Claude Code to fix the bug. When they do that, you record all LLM inputs and outputs.
4. You use these **inputs/outputs** for **supervised finetuning** of your model.
5. You use the **test script result** (True/False) for **reinforcement learning**.

Supervised finetuning trains the conversational part of the problem solving. Reinforcement learning trains the actual problem-solving part.

This way, you don't train a weaker student. Verifiable results (True/False) combined with a solution space exploration technique is what eventually makes the student stronger than the teacher.

Coding LLMs is a solved problem and everyone will end up having the same solution.

[By Andriy Burkov]

Source: https://www.linkedin.com/feed/update/urn:li:activity:7474896006208323584/

--- 

## Analysis: Is this distillation?

Partly, but the author is conflating a few distinct things, and "distillation" only cleanly describes one of his five steps.

What he describes in steps 3–4 (recording the teacher's inputs/outputs and using them for supervised finetuning) is essentially distillation — training a student on a stronger teacher's behavior. Notably it's behavioral/trajectory distillation on the teacher's generated tokens (reasoning traces and tool-call sequences), not the soft-logit distillation of the classic definition. That matches how modern LLM distillation actually works.

But step 5 — the reinforcement learning against the verifiable True/False test result — is not distillation. That's RL with a verifiable reward (RLVR), and it's the part he himself says makes the student exceed the teacher. That's the key tell: pure distillation caps the student at roughly the teacher's level (you're regressing toward the teacher's distribution). His own claim that the student becomes stronger than the teacher only holds because of the RL-against-ground-truth component, where the reward signal comes from the test script, not from any teacher model. The teacher there is just generating a curriculum of verifiable problems, not labels to imitate.

So a precise reading: the pipeline uses distillation for the "conversational"/format part and RLVR for the actual problem-solving capability. The author gestures at distillation but is really describing a hybrid SFT-warmstart-plus-RL recipe. Calling the whole thing "distillation" would be the imprecise part — the strength gain he's excited about specifically comes from the non-distillation half.

One caveat worth flagging: his framing that the American labs "provide" this data assumes their tool outputs are freely recordable and usable for training, which is generally against those providers' terms of service — so the "all scripted, no human in the loop" picture glosses over that friction.