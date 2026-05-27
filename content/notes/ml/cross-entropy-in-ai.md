---
title: Cross-Entropy in AI
description: Explanation of cross-entropy as a loss function in AI, including intuition, formal definition, examples, and relationship to entropy and KL divergence
tags:
  - notes
date: "2026-04-25T16:47:31.262675Z"
lastmod: "2026-04-25T22:19:15+05:30"
draft: false
---
## Cross-Entropy in AI
**Cross-entropy** is a concept from Information Theory that is widely used in **machine learning** to measure how different two probability distributions are.
In AI, it is most commonly used as a **loss function** to evaluate how well a model’s predicted probabilities match the actual (true) labels.
---
## 🧠 Intuition
Think of cross-entropy as a way to answer:
> *“How surprised is the model when it sees the true answer?”*
- If the model assigns **high probability** to the correct answer → low surprise → **low loss**
- If the model assigns **low probability** to the correct answer → high surprise → **high loss**
---
## 📐 Formal Definition
For a true distribution **P** and predicted distribution **Q**, cross-entropy is:

H(P, Q) = - Σ P(x) * log(Q(x))

### In classification (simplified case):
If the true label is one-hot encoded:

Loss = -log(predicted_probability_of_true_class)

---
## 🔍 Example
Suppose you’re doing a classification task with 3 classes:
- True label: `[0, 1, 0]` (Class 2 is correct)
### Case 1: Good prediction

Predicted: [0.1, 0.8, 0.1]
Loss = -log(0.8) ≈ 0.22  (low)

### Case 2: Bad prediction

Predicted: [0.7, 0.2, 0.1]
Loss = -log(0.2) ≈ 1.61  (high)

👉 The worse the prediction, the higher the loss.
---
## ⚙️ Why Cross-Entropy is Used
- Works naturally with **probabilities**
- Strongly penalizes **confident wrong predictions**
- Differentiable → ideal for **gradient-based optimization**
- Pairs well with **softmax** in classification models
---
## 🔗 Relationship to Other Concepts
- **Entropy**: Measures uncertainty in a distribution  
- **Cross-Entropy**: Measures mismatch between two distributions  

KL(P || Q) = CrossEntropy(P, Q) - Entropy(P)

---
## 🧩 Where You’ll See It
- Classification models (e.g., logistic regression, neural networks)
- Language models (predicting next word probabilities)
- Image classification tasks
- Any probabilistic prediction system
---
## 🧭 Quick Summary
- Cross-entropy measures **how wrong** a predicted probability distribution is
- Lower is better
- It’s the **default loss function** for most classification problems in AI