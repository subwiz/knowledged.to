---
title: 'Load-Bearing: Meaning in Agent Memory'
description: Defines “load-bearing” as an active constraint that still supports a decision, memory, comment, or workaround.
tags:
  - definitions
  - memory
  - terminology
date: "2026-05-28T10:27:19Z"
lastmod: "2026-05-28T10:28:00Z"
draft: false
---

"Load-bearing" means something that is actively holding up a decision or assumption — if you removed it, things would change or break.

## Origin in agent memory context

The phrase appears in Claude Code memory-writing guidance:

> Project memories decay fast, so the why helps future-you judge whether the memory is still **load-bearing**.

## What it means in practice

A memory (or comment, or piece of code) is load-bearing if the reason behind it still applies. If the underlying constraint — a deadline, a stakeholder ask, an incident, an invariant — is gone, the memory is no longer load-bearing. It becomes historical trivia, and acting on it can mislead.

## Example

- "Do not merge to main after Thursday — mobile release cut" is load-bearing **until** the release ships.
- After the release, the same sentence is dead weight and should be removed or it will cause confusion the next time someone reads it.

## How to use the concept

When reviewing a memory, comment, flag, or workaround, ask:

1. What constraint does this exist to satisfy?
2. Is that constraint still active?
3. If not, delete it — do not preserve it for sentimental or audit reasons.

This is the same test that distinguishes a real invariant from cruft in any long-lived codebase or knowledge base.