---
title: Primacy Bias in LLM Style Selection
description: Explanation of primacy bias in LLM selector prompts, how alphabetical candidate ordering caused over-selection of certain styles in BHQ, and fixes using deterministic non-lexicographic shuffling
tags:
  - notes
date: "2026-04-08T21:43:53+05:30"
lastmod: "2026-04-08T21:43:53+05:30"
draft: false
---
# Primacy Bias in LLM Style Selection

## What primacy bias is

Primacy bias is the tendency of an AI model to give disproportionate weight to items that appear earlier in a list or prompt. When a model is asked to choose from many options, options shown first can become over-represented in the final answer even when later options are equally or more appropriate.

In practical terms, this means that a selector prompt like:

- style-a
- style-b
- style-c
- ...

can systematically prefer `style-a` more often than expected if the candidates are always presented in the same order.

## Why primacy bias happens

The exact internal cause is not always directly observable from application code, but the working theory is well understood:

1. LLMs process prompts sequentially, so early items establish the initial frame for the decision.
2. During long candidate-list tasks, the model may anchor on the first few plausible options before it has fully compared the rest.
3. If the prompt shape is repeated across many requests with stable ordering, the bias compounds into a visible production pattern.
4. When the candidate list is long, the model may perform a shallow satisficing search instead of a full global comparison, which makes early acceptable answers more likely to survive.

This is not the same as a hard-coded rule in the application. It is an emergent prompt-ordering bias that can show up when model selection is driven by long, ordered candidate lists.

## What happened in the BHQ case

After BHQ-2033 moved `analyze_visual_styles` out of `ctxms` and into `agenticms`, style selection began using local cached catalogs plus an LLM selector prompt.

The relevant behavior at that point was:

- style candidates were filtered by playbook compatibility
- the filtered candidates were sorted alphabetically by style ID
- the sorted candidate list was passed to the LLM in that same alphabetical order
- the fallback path also picked the first few candidates from that same sorted list

This created two different bias surfaces:

### 1. Prompt-order bias in the main LLM selector

For the normal selection path, the model was shown candidate styles in alphabetical order. Because `annotated-statement-typography` sorts very early, it often appeared near the top of the prompt. The working theory was that this caused the model to choose it disproportionately often, especially when many candidates were broadly compatible.

### 2. Alphabetical bias in the fallback path

If the LLM call failed or returned invalid JSON, the fallback logic returned the first three styles from the alphabetically sorted list. In that path, `annotated-statement-typography` was not just favored by prompt position; it was guaranteed to be in the fallback set whenever it survived filtering.

## Concrete evidence from this incident

The incident report that became BHQ-2149 was based on repeated observation that `annotated-statement-typography` appeared in multiple consecutive style-selection results.

Code review confirmed:

- candidate ordering was alphabetical before prompt construction
- fallback selection was also alphabetical-first
- `annotated-statement-typography` had broad playbook tags and frequently survived filtering

Additional investigation showed that the style was not uniquely broad, but it was broad enough to stay in the candidate pool for common combinations such as Authority and Transformation. Once it survived filtering, alphabetical ordering pushed it toward the front of the prompt and into the fallback set.

## Fixes applied

Two fixes were introduced in `agenticms`:

1. Candidate order passed to the LLM is no longer alphabetical.
   Instead, it is reordered using a deterministic per-request hash, which removes stable lexicographic primacy while keeping debugging reproducible for the same brief.

2. Fallback selection no longer takes the first three alphabetically sorted candidates.
   It now uses the same deterministic non-lexicographic ordering before choosing the fallback set.

This preserves reproducibility while eliminating the structural alphabetical advantage that certain styles had.

## Practical lesson

Whenever an LLM is asked to choose from a long set of candidates, candidate order is part of the model behavior surface.

If the order is stable and meaningful only to the implementation (for example alphabetical by slug), that order can accidentally become a hidden ranking signal. In production systems, that can look like the model has a "favorite" option when the real issue is deterministic prompt ordering.

## Design guidance

To reduce primacy bias in future selector prompts:

- do not present candidates in lexicographic or insertion order unless that order is semantically meaningful
- prefer deterministic shuffling or another neutral ordering strategy
- ensure fallback logic does not reintroduce the same ordering bias
- log both the candidate count and the chosen items so repeated patterns can be detected early
- audit metadata breadth separately from prompt-order effects; these are related but distinct bias sources
