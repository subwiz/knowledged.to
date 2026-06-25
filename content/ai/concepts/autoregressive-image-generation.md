---
title: Autoregressive Image Generation
description: Explains autoregressive image generation as sequential visual-token prediction using Transformer-style next-token modeling.
tags:
  - autoregressive
  - image-generation
  - generative-ai
  - transformers
  - machine-learning
date: "2026-06-24T10:03:07Z"
lastmod: "2026-06-24T10:03:23Z"
draft: false
---

Autoregressive image generation builds an image **one piece at a time**, where each new piece is predicted based on everything generated so far — exactly the way an LLM writes text one token after another, with each word conditioned on the preceding words. "Autoregressive" just means "predicting the next element from previous elements."

## How it works

**1. Turn the image into a sequence of tokens.** An image is 2D, but autoregressive models work on 1D sequences. So first you need a way to represent an image as a list of discrete tokens (like words in a vocabulary). This is done with a separate component — often called a **visual tokenizer** or VQ (vector-quantized) autoencoder. It compresses the image into a grid of code numbers drawn from a fixed "codebook" of visual patterns. For example, a 256×256 image might become a 16×16 grid = 256 tokens, each token being an index into a vocabulary of, say, a few thousand possible visual "words." A matching decoder can turn that grid of tokens back into pixels.

**2. Flatten the grid into a sequence.** The 2D token grid is read out in some fixed order — typically raster order, left to right, top to bottom — producing a 1D sequence of tokens, just like a sentence.

**3. Predict tokens one at a time.** A Transformer is trained to predict the next token given all previous tokens. At generation time it starts from the conditioning (e.g., your text prompt) and emits token 1, then feeds that back in to predict token 2, then token 3, and so on until the whole grid is filled. Each step the model outputs a probability distribution over the codebook and samples one token — which is why output is varied rather than deterministic.

**4. Decode back to pixels.** Once the full sequence of image tokens is generated, the tokenizer's decoder converts that grid back into an actual image.

For **text-to-image**, the prompt is simply prepended (or cross-attended) so the model is predicting image tokens conditioned on the text — this is also why models like Gemini can mix text and image tokens in one unified sequence, since both are just tokens to the same Transformer.

## How the first prediction works

The first token is the one exception to "predicted based on everything generated so far," because there's nothing generated yet to condition on. A few things make it work:

**It's conditioned on the prompt, not on prior image tokens.** Even though no image tokens exist yet, the model isn't starting from nothing. For text-to-image, the prompt's tokens are already in the sequence. So the "first" image token is predicted from the text — e.g., "an astronaut riding a horse" — which gives the model plenty to go on. The general rule still holds; it's just that at step one, "everything so far" is the conditioning rather than earlier image tokens.

**A start token seeds it.** Sequence models use a special **beginning-of-sequence (BOS) token** — a fixed marker that means "start here." The model always has at least this token to attend to, so the first real prediction is "predict the next token given the BOS token (plus any prompt)." This is the same mechanism an LLM uses to produce the first word of a response.

**It's a probability distribution, then a sample.** The first prediction isn't a single forced answer — the model outputs a probability distribution over the whole codebook of possible visual tokens, and one is sampled from it. With a rich prompt, that distribution is shaped toward tokens consistent with the description. With no prompt at all (unconditional generation), the distribution reflects what the training data tends to start with, and the random sample is what makes each generated image different from the first token onward.

So: the first token is predicted from the conditioning context (BOS marker + prompt) rather than from previous image content — and from the second token on, "everything generated so far" includes the image tokens too.

## Contrast with diffusion

The key contrast with diffusion is the generation strategy. Diffusion starts with the *whole* image as noise and refines all of it simultaneously over many denoising steps. Autoregressive generation instead produces the image *sequentially*, committing to one token (one region) at a time, conditioned on what came before — never revisiting earlier tokens.

The tradeoffs:

The big advantage is that it slots naturally into the LLM paradigm — same Transformer, same next-token objective — so a single model can fluidly handle text and images together, which is what powers tightly integrated multimodal systems. The classic disadvantage is **speed**: generating tokens strictly one after another is sequential and can be slow for high-resolution images (many tokens), whereas diffusion steps process the whole image in parallel. There's also an error-accumulation risk — a bad early token can throw off everything that follows. Modern systems mitigate the speed problem with tricks like predicting multiple tokens at once, hierarchical/coarse-to-fine token orders, or hybrid AR-then-diffusion-refinement schemes.

One nuance: not all "autoregressive" image models go strictly pixel-region by region in raster order — some predict at multiple scales (coarse structure first, then detail), which is closer to how the better modern systems actually behave. But the core idea is the same: generate the image as a sequence of predicted tokens, each conditioned on the ones before it.