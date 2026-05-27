---
title: Unsloth Studio — Fine-tuning Dataset Formats
description: Overview of dataset formats supported by Unsloth Studio for fine-tuning, including JSONL, Alpaca, ShareGPT, ChatML, and Reasoning formats with rules and best practices and dataset size guidelines
tags:
  - ml
  - fine-tuning
  - unsloth
  - dataset
  - jsonl
date: "2026-04-23T16:39:20.696406Z"
lastmod: "2026-04-23T22:15:42+05:30"
draft: false
---

Unsloth Studio supports several dataset formats depending on your fine-tuning goal. Files can be uploaded directly as **JSONL, JSON, CSV, Parquet, PDF, or DOCX**.

---

## Format Overview

### 1. Raw Text (Continued Pretraining)

Used to inject domain knowledge without any structure. The model learns from continuous prose.

```
The mitochondria is the powerhouse of the cell. ATP synthesis occurs via oxidative phosphorylation...
```

Best for: books, articles, documentation dumps, codebases.

---

### 2. Alpaca Format (Single-turn Instruction)

A structured instruction-following format with three fields: `instruction`, `input` (optional), and `output`.

```jsonl
{"instruction": "Summarize what a Temporal workflow is", "input": "", "output": "A Temporal workflow is a durable function that executes reliably across failures, retries, and server restarts."}
{"instruction": "Explain idempotency in the context of billing", "input": "A payment service that charges per API call", "output": "Idempotency ensures that if the same charge request is submitted multiple times, only one charge is processed. This is typically achieved using a unique idempotency key per transaction."}
```

The `input` field is optional — omit it if the instruction is self-contained.

---

### 3. ShareGPT Format (Multi-turn Conversation)

Used for chatbot-style fine-tuning. Uses `"from"` / `"value"` keys, alternating between `human` and `gpt` turns.

```jsonl
{"conversations": [{"from": "human", "value": "What is a distributed transaction?"}, {"from": "gpt", "value": "A distributed transaction spans multiple services or databases, ensuring all-or-nothing semantics across them."}]}
{"conversations": [{"from": "human", "value": "How do retries work in async systems?"}, {"from": "gpt", "value": "Retries re-attempt a failed operation after a delay. They should be combined with idempotency to avoid duplicate side effects, and exponential backoff to reduce load during failures."}]}
```

Unsloth provides a `standardize_sharegpt()` utility to normalize minor variations in this format.

---

### 4. ChatML Format (Multi-turn, OpenAI-style)

The format used by OpenAI and defaulted to by Hugging Face. Uses `"role"` / `"content"` keys.

```jsonl
{"messages": [{"role": "system", "content": "You are a systems engineering expert."}, {"role": "user", "content": "Explain backpressure in message queues."}, {"role": "assistant", "content": "Backpressure is a mechanism where a consumer signals to a producer to slow down when it cannot keep up with the rate of incoming messages."}]}
{"messages": [{"role": "user", "content": "What is the difference between at-least-once and exactly-once delivery?"}, {"role": "assistant", "content": "At-least-once guarantees a message is delivered but may result in duplicates. Exactly-once guarantees no duplicates but requires coordination overhead such as idempotency checks or transactional messaging."}]}
```

Studio auto-maps columns when it detects this structure.

---

### 5. Reasoning Format (Chain-of-Thought / R1-style)

Used when fine-tuning models that should exhibit step-by-step reasoning. The answer includes a `<think>` block followed by the final response.

```jsonl
{"instruction": "Is this API design idempotent?", "input": "POST /charge with body {amount, user_id}", "output": "<think>The endpoint uses POST and takes a user_id and amount. Without an explicit idempotency key, resubmitting the same request could result in double charges.</think>\n\nNo, this design is not idempotent as written. Include an `idempotency_key` field in the request body and check it server-side before processing."}
```

Use this for distilled reasoning models (e.g. DeepSeek-R1 variants). To train a non-reasoning model to gain reasoning ability, use GRPO/RL instead.

---

## JSONL Format Rules

JSONL (JSON Lines) means **one JSON object per line**, with no commas between lines and no outer array.

**Correct JSONL:**
```
{"instruction": "...", "output": "..."}
{"instruction": "...", "output": "..."}
```

**Wrong — this is plain JSON, not JSONL:**
```
[
  {"instruction": "...", "output": "..."},
  {"instruction": "...", "output": "..."}
]
```

The wrapped array format will cause parse errors in Unsloth and the underlying Hugging Face `datasets` loader.

---

## Dataset Size Guidelines

| Dataset Size | Recommended Model |
|---|---|
| 1,000+ rows | Fine-tune the **base** model |
| 300–1,000 rows | Either base or instruct model |
| < 300 rows | Fine-tune the **instruct** model |

Minimum recommended: **100 rows** for reasonable results.

---

## How Studio Handles Column Mapping

If Studio cannot automatically detect the format, a **Dataset Preview dialog** opens where you manually assign columns to roles: `instruction`, `input`, `output`, `image`, etc. Suggested mappings are pre-filled where possible.

---

## Synthetic Dataset Generation (Data Recipes)

If you have unstructured source material (PDFs, CSVs, DOCX), Unsloth Studio's **Data Recipes** feature (powered by NVIDIA NeMo DataDesigner) can automatically convert it into a training-ready dataset via a visual node workflow. You can also feed the generated data back into an LLM to iteratively improve quality.
