---
title: SWE-bench & SWE-bench Pro Explained
description: Overview of SWE-bench and SWE-bench Pro, the real-world GitHub issue fixing benchmarks used to evaluate AI coding ability.
tags:
  - swe-bench
  - benchmarks
  - llm
  - coding
  - claude
date: "2026-05-16T16:40:54.835578Z"
lastmod: "2026-05-16T22:16:28+05:30"
draft: false
---

**SWE-bench** is a benchmark that tests whether an AI model can actually fix real GitHub issues from open-source Python repositories (like Django, Flask, scikit-learn, etc.). The model is given a repo, a bug report or feature request, and has to produce a code patch that makes the failing tests pass — without being told what to change.

It's considered one of the more meaningful coding benchmarks because it tests end-to-end software engineering ability: reading existing code, understanding context, making targeted changes, and not breaking other things.

**SWE-bench Pro** is a harder variant with:
- More complex, multi-file issues
- Less "solved" training data (so models can't pattern-match from memorized solutions)
- Tasks that require reasoning across larger codebases

**What "64.3% on SWE-bench Pro" means in practice:**
Claude Opus 4.7 successfully resolves ~64 out of every 100 real-world GitHub issues it's given. The remaining ~36 it either gets wrong or doesn't attempt. That's a high bar — these are issues that stumped human developers enough to file a bug report, and the fix has to pass the existing test suite.

**Why it matters for AI app builders:**
If you're using Claude in a coding agent, agentic code review, or automated PR workflow, SWE-bench Pro performance is a reasonable proxy for how well it will handle messy, real-world codebases as opposed to clean textbook problems. A model with a high score here is less likely to produce patches that break unrelated tests or misread the codebase structure.

The short version: it's currently the closest thing the industry has to a "does this model actually write working code?" test.

SWE = Software Engineering. The full name is Software Engineering Benchmark, created by researchers at Princeton and Stanford in 2023.