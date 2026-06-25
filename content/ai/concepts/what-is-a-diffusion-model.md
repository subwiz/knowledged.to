---
title: What is a Diffusion Model?
description: Explains diffusion models as generative AI systems that learn to create data by reversing a noising process.
tags:
  - diffusion-models
  - generative-ai
  - machine-learning
  - deep-learning
date: "2026-06-24T08:32:49Z"
lastmod: "2026-06-24T08:33:04Z"
draft: false
---

A diffusion model is a type of generative AI model that creates new data (most commonly images, but also audio, video, and molecules) by learning to reverse a gradual noising process.

The core idea has two phases:

**Forward process** — You take real training data and progressively add small amounts of random noise over many steps until it becomes pure static, like TV snow. This part is fixed and requires no learning.

**Reverse process** — A neural network is trained to undo that corruption one step at a time: given a noisy input, predict the noise that was added so it can be subtracted. By learning this denoising at every noise level, the model effectively learns the structure of the data.

To generate something new, you start from pure random noise and run the learned reverse process repeatedly. Each step removes a bit of noise until a coherent sample emerges. Because you start from random noise, every run produces a different result.

For text-to-image tools (Stable Diffusion, DALL·E, Midjourney, Imagen), the denoising is *conditioned* on a text prompt, so the network steers the noise removal toward an image matching your description. Most modern systems also run the diffusion in a compressed "latent" space rather than on raw pixels, which makes them much faster.

Compared to earlier approaches like GANs, diffusion models tend to produce higher-quality, more diverse output and train more stably, though generation is slower because it takes many sequential denoising steps. The main intuition: it's easier to teach a model to clean up a slightly noisy image than to paint a perfect one from scratch in a single shot, so you break the hard problem into many easy steps.