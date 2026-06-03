---
title: Model Type Classification by Modality (Multimodal, Vision, Image Generation)
description: Classifies multimodal, vision, and image-generation models by their input/output modalities.
tags:
  - ml
  - multimodal
  - vision
  - image-generation
  - terminology
date: "2026-06-02T17:28:24Z"
lastmod: "2026-06-02T17:28:36Z"
draft: false
---

Classification of model types by modality, focused on the input/output distinction.

## Multimodal

The umbrella term. A model that handles more than one modality (text, images, audio, video) as input and/or output. The term does not specify *which* modalities or in *which* direction — it just means "more than one."

## Vision model

Supports images as **input** — understanding/analyzing them. In ML, "vision" almost always means perception, not generation. These take images in and produce text out (e.g. GPT-4V, Claude with vision). A vision model does **not** generate images.

## Image generation

No single tidy term, but the common labels:

- **Image generation model** / **text-to-image model** — the generic, most-used label (DALL·E, Stable Diffusion, Midjourney, Imagen).
- **Generative vision model** — used occasionally, but ambiguous.
- By architecture: **diffusion models** (most current generators), **GANs** (older), **autoregressive image models**.
- **Any-to-any** / **omni model** — newer term for models that both understand and generate across modalities in one model.

## Useful framing: input → output split

| Direction | Name |
|-----------|------|
| text → image | text-to-image |
| image → text | vision / image understanding |
| image → image | image editing / image-to-image |
| text+image → text+image | multimodal generative / any-to-any |

**Summary:** "Multimodal" is the umbrella; "vision" is specifically the *understanding* side; image generation lives under "text-to-image" or the underlying architecture name (usually diffusion).