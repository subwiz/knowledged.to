---
title: Why ripgrep is faster than grep
description: Explains why ripgrep is often faster than grep via filtering, parallelism, regex literals, and SIMD search.
tags:
  - ripgrep
  - grep
  - performance
  - regex
  - cli-tools
  - simd
date: "2026-06-05T02:57:31Z"
lastmod: "2026-06-05T02:59:56Z"
draft: false
---

ripgrep's speed comes from a combination of architectural and algorithmic choices.

## Smart file traversal and filtering
- Respects `.gitignore` by default and skips hidden and binary files, so it searches far fewer bytes than `grep -r` on a typical project tree. Often the single biggest practical win.
- Directory traversal is parallelized; stat-ing and filtering happen across threads.

## Parallelism
- Searches multiple files concurrently via a thread pool sized to the CPU. Plain `grep` is single-threaded. On multi-file searches this scales almost linearly with cores.

## Regex engine (Rust `regex` crate)
- Finite-automata engine (Thompson NFA / lazy DFA) with guaranteed linear-time matching and no catastrophic backtracking. Compiles the DFA lazily and caches states as it goes.
- Analyzes the pattern to extract required literals (prefixes, suffixes, inner substrings). Instead of running the full regex over every byte, it uses fast literal scanning to find candidate locations, then runs the heavier automaton only near those candidates.

## SIMD-accelerated literal search
- Literal and multi-literal scanning uses vectorized substring search (memchr / Teddy algorithm), checking many bytes per instruction and skipping non-matching regions quickly. GNU grep uses memchr-style tricks too, but ripgrep's literal extraction is more aggressive.

## Memory strategy
- Memory maps for large files; a reusable buffer for streaming/many small files, avoiding per-line allocation. Matching works on raw byte buffers rather than decoded strings.
- Line counting/extraction is deferred — it finds matches first and computes line numbers and boundaries only when there's something to report.

## UTF-8 handling
- Automata operate directly on UTF-8 bytes rather than decoding to codepoints, so Unicode-aware matching imposes no decode pass.

## Nuance
- In a pure single-file, single-threaded, no-ignore-rules benchmark, GNU grep is competitive and sometimes wins — both lean on memchr and Boyer-Moore-style skipping. ripgrep's large real-world advantage comes mostly from parallelism plus searching less data via smart filtering.

Reference: Andrew Gallant's blog post "ripgrep is faster than {grep, ag, git grep, ucg, pt, sift}".