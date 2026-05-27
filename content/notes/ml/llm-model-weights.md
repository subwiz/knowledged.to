---
title: What Are Model Weights in an LLM?
description: Explanation of what model weights are in LLMs, how they encode learned behaviour, why parameter count matters, and how systems like Ollama load them into memory
tags:
  - notes
date: "2026-04-13T19:17:58+05:30"
lastmod: "2026-04-13T19:17:58+05:30"
draft: false
---
# What Are Model Weights in an LLM?

Model weights are the learned numbers inside the neural network.

During training, the model adjusts billions of numeric parameters so that, given some input text, it becomes better at predicting the next token. Those parameters are the weights.

## Short Intuition

A useful way to think about it:

- The model architecture is the blueprint.
- The weights are the filled-in values that make the blueprint useful.
- Without weights, the model is just an empty structure.

## What Weights Do

Weights control how information flows through the network.

They determine:

- how strongly one internal feature affects another
- which patterns the model has learned from training data
- how the model transforms input tokens into probabilities for the next token

In practice, weights are the model's learned behavior encoded as numbers.

## Why They Matter

When people say a model is "7B" or "70B", they are usually referring to the number of parameters or weights.

More weights often mean:

- more memory usage
- more computation
- potentially stronger modeling capacity

But more weights also mean the model is heavier to load and run.

## In Real Systems Like Ollama

A model file stored on disk mostly contains these learned weights.

When Ollama loads a model, it is mainly loading those numbers into RAM and sometimes VRAM so inference can begin.

That is one major reason the first prompt is often slower: the system has to bring the model's learned parameters into memory before it can generate text.

## Tiny Analogy

If a neural network is a huge board of adjustable knobs, the weights are the positions of those knobs after training.

Training is the process of turning all those knobs until the model becomes good at prediction.
