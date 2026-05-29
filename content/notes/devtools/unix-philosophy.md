---
title: Unix Philosophy
description: 'Summary of Unix philosophy: small composable programs, text streams, pipes, and clear modular interfaces.'
tags:
  - unix
  - software-design
  - philosophy
date: "2026-05-29T14:11:57Z"
lastmod: "2026-05-29T14:18:41Z"
draft: false
---

The core Unix philosophy:

Build small programs that do one thing well, make them work together, and communicate through simple text streams.

Common principles:

* Do one thing well.
* Compose tools using pipes and standard input/output.
* Use text as a universal interface where practical.
* Keep programs simple, modular, and replaceable.
* Prefer clear interfaces over monolithic systems.

Tools like `grep`, `sort`, `uniq`, `awk`, and `sed` are powerful not because each does everything, but because they combine cleanly.