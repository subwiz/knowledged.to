---
title: Recurrent Neural Networks (RNNs)
description: Overview of RNNs, their memory mechanism, common variants (LSTM, GRU), use cases, and how they compare to Transformers
tags:
  - notes
date: "2026-04-06T22:12:16+05:30"
lastmod: "2026-04-06T22:12:16+05:30"
draft: false
---
RNN stands for **Recurrent Neural Network** — a type of neural network designed to work with sequential data.

Unlike standard feedforward networks, RNNs have a "memory" mechanism: they pass information from one step to the next, making them well-suited for tasks where order and context matter, like text, speech, or time-series data.

The key idea is that at each step, the network takes both the current input *and* a hidden state from the previous step, producing an output and an updated hidden state.

**Common variants:**
- **LSTM** (Long Short-Term Memory) — solves the vanishing gradient problem, better at capturing long-range dependencies
- **GRU** (Gated Recurrent Unit) — a simpler, faster alternative to LSTM

**Typical use cases:** language modeling, machine translation, speech recognition, sentiment analysis, and time-series forecasting.

That said, RNNs have largely been superseded by **Transformers** in most NLP tasks, since Transformers handle long-range dependencies more effectively and parallelize much better during training.