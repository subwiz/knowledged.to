---
title: 'Jailbreaking LLMs: A Security Researcher''s Field Guide'
description: Field guide to LLM jailbreaking attack surfaces, threat modeling, defenses, and responsible disclosure.
tags:
  - ai-security
  - llm
  - jailbreak
  - red-team
  - threat-modeling
date: "2026-06-14T15:35:49Z"
lastmod: "2026-06-14T15:36:02Z"
draft: false
---

Jailbreaking is the practice of getting a language model to do something its safety training was built to prevent — emit restricted content, ignore its system prompt, or reveal that prompt. For a red-teamer it's an evaluation problem: not "can I make the model say a bad word," but "can I extract genuine, actionable capability uplift past the core safeguards." Keep that distinction central, because it separates real findings from cosmetic ones.

## Attack surface

Most published techniques cluster into a few families:

- **Instruction-hierarchy confusion** — pitting user input against the system prompt (role-play personas, "ignore previous instructions," fake developer messages). Works when the model can't reliably rank trust levels of competing instructions.
- **Encoding and obfuscation** — Unicode homoglyphs, base64, leetspeak, translation, token-splitting. The goal is to slip a payload past input classifiers while keeping it legible to the model.
- **Context manipulation** — long-context dilution, fictional/narrative framing, and gradual escalation that normalizes a request over many turns.
- **Decomposition** — splitting a prohibited task into individually benign sub-requests, then recomposing the outputs client-side.
- **Prompt extraction** — coaxing the model to reproduce its system prompt or tool definitions, which then enables more targeted attacks.

These compose. The strongest reported breaks layer obfuscation, framing, and decomposition together rather than relying on a single trick.

## Threat modeling

Severity should track real-world impact, not refusal-bypass alone. A useful rubric: Does the output provide meaningful uplift toward a high-consequence harm (bio, chem, large-scale cyber)? Is it reproducible across sessions, or a lucky sample? Does it survive a model or system-prompt update? A one-off transcript where the model "keeps talking" after a refusal is weak evidence; a repeatable pipeline that yields operational detail is a real finding.

## Defensive posture

Defense is layered and probabilistic, not a single gate. Practical controls include input/output classifiers, instruction-hierarchy training so system prompts outrank user text, constitutional/RLHF alignment on the base model, retrieval and tool sandboxing, rate limiting, and abuse monitoring on aggregate behavior rather than single prompts. Assume any one layer can be bypassed and design for graceful degradation.

## Disclosure

Treat jailbreaks like vulnerabilities: reproduce, quantify uplift, and report through the provider's responsible-disclosure channel before publishing. Sharing working exploit chains publicly raises real-world risk without improving the model's safety. The valuable contribution is a clear, reproducible signal that helps defenders close the gap.
