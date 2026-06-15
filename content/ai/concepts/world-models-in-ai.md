---
title: World Models in AI
description: Explains AI world models as internal predictive representations for planning across RL, LLMs, and robotics.
tags:
  - ai
  - world-models
  - reinforcement-learning
  - llms
  - robotics
date: "2026-06-14T16:52:11Z"
lastmod: "2026-06-14T16:52:24Z"
draft: false
---

A "world model" in AI is an internal representation that a system builds of how its environment works — capturing not just what things are, but how they relate, change, and respond to actions. It lets the system predict what will happen next, including the consequences of its own behavior, rather than just reacting to immediate inputs.

The core idea is prediction. A system with a world model can simulate "if X happens, then Y follows" internally, without having to try it in reality. This is roughly analogous to how humans carry a mental model of the physical and social world that lets us anticipate outcomes.

The term shows up in a few overlapping contexts:

## Reinforcement learning

A world model is an explicit, learned model of the environment's dynamics — given a state and an action, it predicts the next state and reward. Agents can then "plan" or train inside this learned simulation instead of the real environment, which is faster and safer. The well-known *World Models* work by Ha and Schmidhuber (2018) and DeepMind's *Dreamer* line of agents are canonical examples.

## Large language models and foundation models

Here "world model" is used more loosely to ask whether a model has acquired an internal, coherent understanding of how the world works — physical causation, object permanence, spatial relationships, the mental states of others — versus merely learning surface-level statistical patterns in text. This is an active and contested research question.

## Robotics and self-driving

World models predict how a scene will evolve (where other cars and pedestrians will move) so the system can plan actions accordingly.

## Key distinction

The key distinction is between a system that maps inputs directly to outputs (purely reactive) and one that maintains an internal model it can query, simulate, and plan against. The latter is generally seen as important for robustness, sample efficiency, reasoning, and generalization to novel situations — which is why world models are considered by many researchers to be a promising path toward more capable and general AI.