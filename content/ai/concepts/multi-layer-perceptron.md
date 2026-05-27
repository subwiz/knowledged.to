---
title: Multi-Layer Perceptron (MLP)
description: Foundational neural network architecture covering perceptrons, layers, activation functions, and backpropagation-based training.
tags:
  - machine-learning
  - neural-networks
  - deep-learning
  - mlp
  - backpropagation
date: "2026-05-17T06:09:27.596311Z"
lastmod: "2026-05-17T11:39:38+05:30"
draft: false
---

A **Multi-Layer Perceptron (MLP)** is one of the foundational types of artificial neural network. It learns to map inputs to outputs by passing data through a series of layers of interconnected nodes ("neurons"), adjusting internal weights during training until its predictions improve.

---

## Background: The Single Perceptron

To understand an MLP, start with its building block — the **perceptron** (single neuron):

- It takes several numerical inputs $x_1, x_2, \ldots, x_n$.
- Each input is multiplied by a learned **weight** $w_i$ (how important that input is).
- The results are summed, a **bias** term $b$ is added (a constant that shifts the output), and the total is passed through an **activation function** $f$ to produce an output.

$$\text{output} = f\!\left(\sum_{i} w_i x_i + b\right)$$

A single perceptron can only learn **linearly separable** patterns — i.e., problems whose decision boundary is a straight line (or hyperplane). Real-world problems are rarely that simple.

---

## What Makes It "Multi-Layer"?

An MLP stacks multiple layers of perceptrons:

| Layer | Role |
|---|---|
| **Input layer** | Receives raw features (e.g., pixel values, numbers). No computation here. |
| **Hidden layer(s)** | One or more intermediate layers that learn abstract representations. This is where the real learning happens. |
| **Output layer** | Produces the final prediction (e.g., a class probability or a continuous value). |

The layers between input and output are called **hidden** because their values are not directly observed in the data.

```
Input Layer     Hidden Layer     Output Layer
   x1  ──┐
          ├──► [neuron] ──┐
   x2  ──┤                ├──► [neuron] ──► ŷ
          ├──► [neuron] ──┘
   x3  ──┘
```

---

## Activation Functions

Each neuron applies an **activation function** to introduce *non-linearity* — without this, stacking layers would still only produce a linear model, no matter how deep. Common choices:

- **ReLU** (Rectified Linear Unit): $f(x) = \max(0, x)$ — most widely used in hidden layers today.
- **Sigmoid**: $f(x) = \frac{1}{1+e^{-x}}$ — squashes output to (0, 1); used in binary classification outputs.
- **Softmax**: Generalises sigmoid to multiple classes; used in multi-class output layers.
- **Tanh**: Squashes output to (−1, 1); sometimes used in hidden layers.

---

## How an MLP Learns: Backpropagation

Training an MLP means finding the weights that minimise prediction error. This is done by:

1. **Forward pass** — feed an input through the network to get a prediction $\hat{y}$.
2. **Compute loss** — measure how wrong the prediction is using a **loss function** (e.g., Mean Squared Error for regression, Cross-Entropy for classification).
3. **Backward pass (backpropagation)** — use the **chain rule** of calculus to compute how much each weight contributed to the error, producing a *gradient* for every weight.
4. **Weight update** — adjust every weight slightly in the direction that reduces the error, using an optimiser like **Stochastic Gradient Descent (SGD)** or **Adam**.

This cycle repeats over many **epochs** (full passes through the training data) until the loss is acceptably low.

---

## A Concrete Example

Suppose you want to classify whether an email is spam (1) or not (0) using two features: word count and exclamation-mark count.

- **Input layer**: 2 neurons (one per feature).
- **Hidden layer**: 4 neurons with ReLU activation.
- **Output layer**: 1 neuron with Sigmoid activation → outputs a probability between 0 and 1.

During training, the MLP learns weights that combine word count and exclamation marks in a non-linear way to separate spam from non-spam.

---

## Key Properties & Limitations

| Property | Detail |
|---|---|
| **Universal approximation** | An MLP with at least one hidden layer and a non-linear activation can approximate *any* continuous function (given enough neurons). |
| **Fully connected** | Every neuron in one layer connects to every neuron in the next — hence also called a **fully connected network** or **dense network**. |
| **Scalability** | MLPs struggle with raw images or sequences; specialised architectures (CNNs for images, RNNs/Transformers for sequences) usually outperform them there. |
| **Overfitting** | With many parameters, MLPs can memorise training data. Regularisation techniques like **dropout** (randomly zeroing neurons during training) and **weight decay** help. |

---

## Summary

An MLP is a feedforward neural network with:
- An **input layer**, one or more **hidden layers**, and an **output layer**.
- **Non-linear activation functions** enabling it to learn complex patterns.
- Trained via **backpropagation** and **gradient descent**.

It is often the first neural network architecture to learn, and understanding it well forms the foundation for studying deeper and more specialised architectures.