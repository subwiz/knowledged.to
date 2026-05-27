---
title: Model Quantization
description: Overview of model quantization techniques, precision levels, and trade-offs for reducing neural network memory and improving inference speed
tags:
  - notes
date: "2026-04-06T21:39:37+05:30"
lastmod: "2026-04-06T21:39:37+05:30"
draft: false
---
Model quantization is the process of reducing the numerical precision of a neural network's weights (and sometimes activations) to make models smaller and faster, with acceptable loss in accuracy.

## The core idea

Neural networks store parameters as floating-point numbers — typically 32-bit floats (`float32`). Quantization maps these to lower-precision representations like 16-bit floats, 8-bit integers, or even 4-bit integers. Fewer bits per number means less memory and faster arithmetic.

## Common precision levels

| Format | Bits | Typical use |
|---|---|---|
| `float32` | 32 | Training baseline |
| `bfloat16` / `float16` | 16 | Training & inference on GPUs |
| `int8` | 8 | Efficient inference |
| `int4` / `int3` / `int2` | 4 or less | Aggressive compression (LLMs) |

## How it works

**Post-training quantization (PTQ)** takes a trained model and converts its weights after the fact. It's fast and simple but can hurt accuracy at very low bit depths.

**Quantization-aware training (QAT)** simulates low-precision arithmetic during training, so the model learns to be robust to quantization error. This produces better accuracy but requires a full training run.

**The mapping process** works roughly like this: given a range of float values, you find the min/max, divide the range into discrete steps, and map each float to the nearest step. A scale factor and zero-point are stored per tensor (or per channel) to reverse the mapping during computation.

## Why it matters for LLMs

Large language models have billions of parameters. A 70B parameter model in `float32` would require ~280 GB of memory — far beyond a single GPU. Quantizing to `int4` brings that down to ~35 GB, making local inference feasible. Techniques like **GGUF** (used by llama.cpp) and **GPTQ/AWQ** are purpose-built for LLM quantization with minimal perplexity degradation.

## The trade-offs

- **Memory** — fewer bits means the model fits in less RAM/VRAM
- **Speed** — integer arithmetic is faster than floating point on most hardware; also more data fits in cache
- **Accuracy** — lower precision introduces rounding error; some layers (like the first and last) are more sensitive and are often kept at higher precision
- **Outliers** — transformer activations can have extreme outlier values that make quantization harder; methods like SmoothQuant and GPTQ account for this

## A mental model

Think of it like image compression: a raw photo has full color depth, but a compressed JPEG still looks fine at a fraction of the size. Quantization does the same for model weights — you're trading a bit of fidelity for a lot of practical efficiency.