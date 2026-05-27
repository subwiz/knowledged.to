---
title: 'AI Prompts: System Prompt and Other Types'
description: Overview of the different types of AI prompts including system, user, few-shot, zero-shot, chain-of-thought, meta, and retrieval-augmented prompts
tags:
  - notes
date: "2026-04-16T22:42:26+05:30"
lastmod: "2026-04-16T22:42:26+05:30"
draft: false
---

## System Prompt

A **system prompt** is a set of instructions given to an AI model *before* any conversation begins. It's written by the developer or application builder (not the end user) and sets the AI's behavior, persona, tone, rules, and constraints for the entire session. The user typically doesn't see it.

Think of it like a job briefing you give an employee before they meet a customer — it shapes how they behave without the customer knowing the specifics.

For example, a system prompt might say: *"You are a helpful customer support agent for Acme Corp. Only answer questions about our products. Always be polite and concise."*

---

## Other Types of Prompts in AI

**User Prompt** — This is the message the end user actually types. It's the direct question or request sent to the AI in real time. Most people interact only at this level.

**Assistant Prompt (or AI turn)** — In multi-turn conversations, the AI's previous responses can be included as part of the conversation history, effectively "prompting" the next response. This is how context is maintained across a chat.

**Few-shot Prompt** — Examples embedded in a prompt to teach the model a pattern. Instead of just explaining a task, you show the model 2–3 input/output examples and ask it to follow suit.

**Zero-shot Prompt** — Asking the model to do something with no examples at all, relying purely on its training. "Translate this sentence to French."

**Chain-of-Thought Prompt** — A technique where you instruct the model to *think step by step* before giving a final answer, which improves reasoning on complex tasks.

**Meta-prompt** — A prompt whose purpose is to generate or improve other prompts. Useful when you want the AI to help you craft better instructions.

**Retrieval-Augmented Prompt** — A prompt that includes relevant context retrieved from an external database or document at runtime, so the model can answer questions grounded in specific, up-to-date information.

---

In practice, most production AI applications layer several of these together — a system prompt sets the rules, retrieved documents provide context, and the user prompt drives the actual query.