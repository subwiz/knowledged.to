---
title: Instruction Tuning
description: Overview of instruction tuning, how it works, dataset construction, and variants like RLHF, RLAIF, and DPO
tags:
  - notes
date: "2026-04-06T21:54:49+05:30"
lastmod: "2026-04-06T21:54:49+05:30"
draft: false
---
Instruction tuning is a fine-tuning technique where a pre-trained language model is further trained on a dataset of (instruction, response) pairs to make it better at following natural language instructions.

## How it works

A base language model trained on raw text is good at predicting the next token, but not necessarily at being *helpful*. Instruction tuning bridges that gap by showing the model thousands to millions of examples like:

- **Instruction:** "Summarize this article in 3 bullet points."
- **Response:** "• Point 1 …"

The model learns to map user intent → useful output.

## Key ideas

**Dataset construction** — Examples cover a wide range of tasks: summarization, translation, Q&A, coding, reasoning, creative writing, etc. Diversity is crucial so the model generalizes rather than overfits to a narrow task type.

**Format** — Each example typically has a system prompt, a user instruction, and the expected assistant response. This is why models respond well to the chat-style format you're using right now.

**Scale matters** — Research (e.g., FLAN, InstructGPT) showed that even a relatively small number of high-quality instruction examples can dramatically improve a model's ability to generalize to *unseen* instructions.

## Variants worth knowing

| Technique | What it adds |
|---|---|
| **RLHF** (Reinforcement Learning from Human Feedback) | Human raters rank responses; a reward model is trained on those rankings and used to further fine-tune |
| **RLAIF** | Same idea but using AI feedback instead of human raters |
| **Direct Preference Optimization (DPO)** | Skips the reward model; optimizes preferences directly, simpler to train |

## Why it matters

Before instruction tuning, getting useful output from a large model required careful prompt engineering and the model still often "completed" your prompt rather than "answering" it. Instruction tuning is what makes models feel like assistants rather than autocomplete engines.

GPT and Claude are a product of this kind of training pipeline — constitutional AI and RLHF-style techniques built on top of a pre-trained base model.