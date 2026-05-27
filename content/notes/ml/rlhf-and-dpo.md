---
title: 'RLHF and DPO: Aligning AI to Human Preferences'
description: Comparison of RLHF and DPO alignment techniques, covering their pipelines, strengths, weaknesses, and where each is used in practice
tags:
  - notes
date: "2026-04-06T22:03:50+05:30"
lastmod: "2026-04-06T22:03:50+05:30"
draft: false
---
## RLHF and DPO: Aligning AI to Human Preferences

Both techniques address the same core problem: after pre-training on raw text, a language model needs to be *steered* toward responses that are helpful, safe, and aligned with what humans actually want. They're two different approaches to the same goal.

---

### RLHF — Reinforcement Learning from Human Feedback

**The idea:** Train a separate model to *predict what humans prefer*, then use that model as a reward signal to fine-tune the LLM via RL.

**The pipeline:**

1. **Supervised Fine-Tuning (SFT):** Start with the base LLM and fine-tune it on a curated set of high-quality prompt-response pairs to get a reasonable baseline.
2. **Reward Model Training:** Human annotators are shown pairs of model responses and asked *which one is better*. These preferences train a separate "reward model" (RM) that learns to score any response.
3. **RL Optimization:** The LLM is then optimized using PPO (Proximal Policy Optimization) — an RL algorithm — to generate responses that maximize the reward model's score, while a KL-divergence penalty keeps it from drifting too far from the SFT baseline.

**Strengths:**
- Proven at scale (used by InstructGPT, early ChatGPT, Gemini)
- Can capture nuanced human preferences

**Weaknesses:**
- Complex, brittle pipeline — three separate models to train
- PPO is notoriously unstable and compute-intensive
- Reward hacking: the LLM can learn to "game" the reward model without actually being better
- Requires significant infrastructure and careful tuning

---

### DPO — Direct Preference Optimization

**The idea:** Skip the reward model entirely. Mathematically reformulate the RLHF objective so the LLM itself *is* the reward model — optimized directly from preference data.

**The insight:** The optimal policy under the RLHF objective has a closed-form relationship to the reward function. DPO (Rafailov et al., 2023) exploits this to derive a loss function that works directly on preference pairs (chosen vs. rejected responses), without ever training a separate RM or running RL.

**The pipeline:**

1. **SFT baseline** (same as RLHF)
2. **Preference data** — the same human-labeled pairs (chosen/rejected), but fed directly into a classification-style loss on the LLM

**The loss function intuitively:** Increase the relative likelihood of the *chosen* response over the *rejected* one, scaled by how confidently the reference model (SFT baseline) distinguishes them.

**Strengths:**
- Much simpler — one training stage, standard supervised loss
- More stable training, lower compute cost
- No reward hacking (no separate RM to game)
- Increasingly competitive with RLHF at smaller scales

**Weaknesses:**
- Requires high-quality offline preference data (no online exploration)
- Can underperform RLHF on very complex tasks where exploration matters
- Sensitive to the quality and distribution of the preference dataset

---

### Side-by-Side Comparison

| | **RLHF** | **DPO** |
|---|---|---|
| Reward model | Explicit, separately trained | Implicit, baked into LLM loss |
| Training algorithm | PPO (RL) | Supervised cross-entropy-style loss |
| Complexity | High (3 models, RL loop) | Low (1 model, 1 loss) |
| Stability | Fragile | Stable |
| Compute cost | High | Lower |
| Online exploration | Yes | No (offline only) |
| Used by | InstructGPT, early ChatGPT | Llama 3, Mistral, many open models |

---

### Where Things Stand

DPO has become the dominant approach in open-source alignment (Llama 3, Mistral, Phi, etc.) because of its simplicity. However, frontier labs (OpenAI, Google DeepMind) still use variants of RLHF — often combining offline DPO-style methods with online RL for the best of both worlds. Hybrid approaches like **RLHF with DPO initialization** or **online DPO** are active research areas.