---
title: Knowledge Distillation
description: Overview of knowledge distillation, how student models learn from teacher model outputs, and applications including edge deployment, speculative decoding, and LLM training
tags:
  - notes
date: "2026-04-06T22:43:20+05:30"
lastmod: "2026-04-06T22:43:20+05:30"
draft: false
---
Distillation in AI (also called **knowledge distillation**) is a model compression technique where a smaller "student" model is trained to mimic the behavior of a larger, more capable "teacher" model.

**How it works**

Instead of training the student on hard labels (e.g., "this image is a cat"), the student learns from the teacher's *soft outputs* — the probability distribution the teacher assigns across all classes. These soft outputs carry richer information. For example, knowing a model thinks an image is 70% cat, 20% leopard, and 10% tiger tells the student more about the underlying structure than just "cat."

**Why it matters**

Large models are expensive to run. Distillation lets you compress their "knowledge" into a smaller model that:
- Is faster and cheaper to serve
- Uses less memory
- Often performs surprisingly close to the original

**Common applications**

- **Edge deployment** — running models on phones or IoT devices
- **SpecDecoding** — a large model verifies outputs from a smaller draft model to speed up inference
- **LLM training** — newer, smaller models trained on outputs from larger frontier models (e.g., DeepSeek's R1 distilled variants were trained on reasoning traces from a larger model)
- **Task-specific compression** — fine-tuning a general large model into a small specialist

**A nuance worth knowing**

There's a distinction between distillation from *logits* (the raw probability outputs) versus distillation from *reasoning traces or chain-of-thought* — the latter is more common in modern LLM work, where the student learns to replicate the teacher's step-by-step reasoning rather than just final token probabilities.

In short: distillation is how the AI field takes big expensive models and squeezes their capabilities into small, deployable ones.