---
title: Orthogonal Work
description: Defines orthogonal work as unrelated changes outside task scope and why to keep diffs surgical.
tags:
  - engineering
  - scope
  - code-review
date: "2026-05-28T14:49:58Z"
lastmod: "2026-05-28T14:50:45Z"
draft: false
---

"Orthogonal work" means changes that are unrelated to the task at hand — they sit on a different axis from what you were asked to do.

## Example

If you are asked to fix a bug in auth, then:

- **On-axis (do it):** the bug fix itself, plus tests for that fix.
- **Orthogonal (do not do it):** renaming variables in the same file, "cleaning up" a nearby function, updating comments you noticed were stale, deleting code that looks unused, reformatting imports.

## Why avoid it

The orthogonal changes might be improvements, but they:

- Bloat the diff and make review harder.
- Mix concerns — if the bug fix needs to be reverted, the unrelated cleanup goes with it.
- Expand the blast radius of a change beyond what was reviewed for.

## Rule of thumb

Surgical precision: touch only what the task requires. Flag the other stuff separately (e.g. as a follow-up task or a spawned chore) rather than bundling it in.