---
title: Model Drift
description: Overview of model drift, detection, mitigation, and LLM-specific issues like knowledge staleness and provider drift.
tags:
  - ml
  - model-drift
  - llm
  - observability
  - mlops
  - rlhf
  - rag
date: "2026-05-21T15:33:36.56108Z"
lastmod: "2026-05-21T21:04:12+05:30"
draft: false
---

Model drift is the general phenomenon where a deployed model's predictive performance degrades over time, even though nothing about the model itself has changed. The model is the same; the world it operates in isn't.

## Taxonomy

Drift is usually classified by what's shifting in the underlying probability distributions.

### Data drift (covariate shift)
The distribution of input features `P(X)` changes, but the relationship `P(Y|X)` stays the same. A fraud detection model starts seeing a higher fraction of mobile-wallet payments — inputs look different, but the rules for "is this fraud" haven't changed.

### Concept drift
The more dangerous case: `P(Y|X)` itself changes. The same input now maps to a different correct output. Spam classifiers face this constantly — spammers adapt, so features that signaled spam two years ago no longer do. A credit-risk model built before COVID had its `P(default | income, employment)` relationship rewritten by the pandemic.

### Label drift (prior shift)
`P(Y)` changes — the base rate of the target variable shifts. During a recession, the base rate of loan defaults rises regardless of any individual borrower's profile.

### Temporal patterns
- **Sudden** — a market crash, a policy change
- **Gradual** — slow demographic or behavioral evolution
- **Incremental** — many small changes compounding
- **Recurring/seasonal** — retail patterns around Black Friday, flu cases in winter

## Detection

Production detection combines statistical tests on inputs/outputs with direct performance monitoring against ground truth (when it eventually arrives).

### Input distribution tests
- **Kolmogorov-Smirnov** — continuous features
- **Chi-squared** — categorical features
- **Population Stability Index (PSI)** — a workhorse in finance ML; interpretable and stable
- **KL divergence, Jensen-Shannon, Wasserstein distance** — compare full distributions rather than summary statistics

### Output/performance monitoring
- Prediction confidence distributions
- Calibration curves
- Lagged accuracy/precision/recall once labels arrive
- **Embedding-based drift detection** — project incoming data through the model's embedding layer and run distribution tests in that space; catches semantic shifts that raw feature stats miss

The hard part isn't the statistical test, it's setting a threshold sensitive enough to catch real degradation but not so jumpy that you retrain on noise.

## Mitigation strategies

| Strategy | How it works | Tradeoff |
|---|---|---|
| Scheduled retraining | Cron-driven, periodic | Predictable, may retrain too often or too rarely |
| Trigger-based retraining | Kicks off when drift metrics cross a threshold | Reactive, depends on threshold tuning |
| Online/continual learning | Incremental updates as data streams in | Catastrophic forgetting risk |
| Champion-challenger | Challenger trains on recent data in shadow; promoted if it beats champion on holdout | Operationally clean, doubles training cost |

**Catastrophic forgetting**: when continual learning makes the model lose old capabilities as it absorbs new ones.

Scheduled retraining is the default for a reason — predictable and easy to reason about.

## LLM-specific drift

For large language models, drift takes specific forms distinct from the classical taxonomy.

### Knowledge staleness
A model with a Jan 2024 training cutoff doesn't know about post-cutoff events. Technically not drift — the model didn't degrade, the world moved on — but the user-perceived effect is the same. Standard mitigation is RAG over a continuously updated corpus rather than retraining the base model.

### Behavioral drift across versions
Chen, Zou, and Zaharia (Stanford/Berkeley, 2023): *"How Is ChatGPT's Behavior Changing over Time?"* documented measurable shifts in GPT-4's behavior on identical prompts across a few months. Some tasks improved, others regressed. Likely culprits: successive RLHF rounds, safety fine-tuning, inference-stack changes — not base weights "decaying" on their own.

### Alignment tax
Each RLHF or safety pass tends to slightly degrade some capabilities (creative writing, instruction-following on edge cases) in exchange for behavioral gains. Over many iterations this compounds. Part of why users perceive models as "getting worse" even when benchmark scores improve.

### Mode collapse / diversity loss
RLHF over-converges on a narrow style — outputs become more uniform, hedged, and predictable, even if individual responses are higher-quality on average.

## Operationally relevant flavors for production LLM stacks

For Go services hitting hosted LLMs with user-facing SLAs and billing on top, three matter most:

1. **Knowledge staleness** — managed via RAG and fresh retrieval
2. **Prompt drift** — user inputs evolve as people get savvier with the product
3. **Provider-side behavioral drift** — if you use a hosted model whose weights you don't control, the provider can silently change behavior

Mitigation for (3): ship a regression eval suite against your LLM provider as part of CI. A fixed set of `(prompt, expected behavior)` pairs running nightly catches silent provider changes before users do. With OTel-heavy observability, treat eval scores as another time-series metric alongside latency and error rate.

## References

- Chen, Zou, Zaharia (2023). *How Is ChatGPT's Behavior Changing over Time?* — Stanford/Berkeley.
- Population Stability Index — standard in credit risk monitoring; see finance ML literature.
