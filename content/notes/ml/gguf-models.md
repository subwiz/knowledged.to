---
title: GGUF Models
description: Overview of the GGUF binary format for storing and distributing LLMs locally, including quantization levels, key characteristics, and popular runtimes like llama.cpp and Ollama
tags:
  - notes
date: "2026-04-10T09:11:52+05:30"
lastmod: "2026-04-10T09:11:52+05:30"
draft: false
---

GGUF (GPT-Generated Unified Format) is a binary file format for storing and distributing large language models, designed specifically for efficient local inference.

## Background

Introduced by the **llama.cpp** project in 2023 as a replacement for the older GGML format. The name reflects its origins but it's now used broadly across many model families beyond GPT.

## Key Characteristics

**Self-contained** — A single `.gguf` file bundles everything needed to run a model: weights, tokenizer vocabulary, metadata, and architecture config. No separate config files needed.

**Quantization-friendly** — GGUF is the go-to format for quantized models. Common quantization levels include Q4_K_M, Q5_K_M, Q8_0, etc. Quantization reduces model size and memory requirements by lowering numerical precision (e.g., from 32-bit floats to 4-bit integers), with varying tradeoffs in quality.

**CPU + GPU inference** — Unlike formats optimized purely for GPU (like safetensors in training pipelines), GGUF models can run efficiently on CPU, with optional GPU offloading for layers that fit in VRAM.

**Metadata-rich** — The format includes a structured key-value metadata section describing the architecture, context length, rope scaling, and more — making it easier for runtimes to load models correctly without external config.

## Why It Matters

It's the dominant format for running open-weight models locally (Llama, Mistral, Phi, Gemma, etc.) using tools like:
- **llama.cpp** — the reference runtime
- **Ollama** — wraps llama.cpp for a Docker-like local model experience
- **LM Studio** — GUI for running GGUF models
- **Jan** — another local inference UI

## Typical Filename Anatomy

A filename like `Meta-Llama-3-8B-Instruct.Q4_K_M.gguf` tells you:
- Model family and size: Llama 3, 8B parameters
- Variant: Instruct-tuned
- Quantization: Q4_K_M (4-bit, K-quant, medium quality)

It's essentially the standard packaging format for the local LLM ecosystem.