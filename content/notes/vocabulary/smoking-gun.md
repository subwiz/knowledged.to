---
title: Smoking Gun in Debugging
description: Defines “smoking gun” as direct, conclusive debugging evidence that pinpoints a problem’s cause.
tags:
  - debugging
  - terminology
  - glossary
date: "2026-05-28T14:45:10Z"
lastmod: "2026-05-28T14:46:02Z"
draft: false
---

"Smoking gun" is a debugging term for **direct, conclusive evidence that pinpoints the cause of a problem** — not a hunch or correlation, but proof.

## Examples

- A stack trace showing the exact line that threw the exception
- A log entry showing `user_id=null` right before the NPE
- A git blame revealing the commit that flipped the config value
- A network trace showing the request was sent with the wrong header

## Contrast with circumstantial evidence

- "Errors started around the time of the deploy" → suggestive, not a smoking gun
- "The deploy changed the auth middleware, and the failing requests all hit that middleware with a malformed token" → smoking gun

## Why it matters

In systematic debugging, you keep digging until you have a smoking gun before fixing — otherwise you're guessing and might patch a symptom instead of the cause.