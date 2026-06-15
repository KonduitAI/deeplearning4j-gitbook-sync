---
title: "Beginner's Guide"
description: "Getting started with deep learning and Deeplearning4j — prerequisites, learning path, and recommended resources"
---

## Getting Started with Deeplearning4j

This guide is for people who are new to deep learning, new to Deeplearning4j, or both. It maps out what you need to know, in what order, and points you to the best free resources to fill each gap.

Deep learning is applied mathematics meeting software engineering. The prerequisites differ depending on whether you want to understand how it works or just use it in an application. Both are valid goals, and the roadmap below covers both.

---

## Do You Need to Understand the Math?

**If you want to use DL4J to build applications:** You can start with the [Quickstart](quickstart.md) after you are comfortable with Java and Maven. The DL4J API is designed to be used without knowing how backpropagation works internally. You configure a network, call `fit`, and call `output`.

**If you want to choose architectures, debug training problems, and tune hyperparameters:** You need a working knowledge of linear algebra (matrix multiplication, dot products, vector spaces) and calculus (gradients, the chain rule). Probability and statistics help for understanding loss functions and evaluation metrics. You do not need a PhD — undergraduate-level familiarity with these topics is sufficient.

---

## Prerequisites

### 1. Java

DL4J is a Java library. You should be comfortable with Java before using it. The most important concepts to know:

- Object-oriented programming (classes, interfaces, inheritance)
- Collections (List, Map) and generics
- Exception handling
- Maven or Gradle for dependency management
- Using an IDE such as IntelliJ IDEA

If you are new to Java, these resources can help:

- [Think Java (interactive)](https://books.trinket.io/thinkjava/) — free, browser-based Java environment
- [Learn Java the Hard Way](https://learnjavathehardway.org/)
- [Head First Java](http://www.amazon.com/gp/product/0596009208) — highly readable book for beginners
- [Intro to Programming in Java at Princeton](http://introcs.cs.princeton.edu/java/home/)
- [JShell in 5 Minutes](https://dzone.com/articles/jshell-in-five-minutes) — experiment with Java without a full project setup

### 2. Linear Algebra

The core data structure in DL4J is the N-dimensional array (NDArray). Understanding how matrices and vectors work will help you debug shape errors, understand layer configurations, and interpret model outputs.

Key topics: vectors, matrices, matrix multiplication, transpose, dot product, norms.

- [Andrew Ng's 6-Part Linear Algebra Review](https://www.youtube.com/playlist?list=PLnnr1O8OWc6boN4WHeuisJWmeQHH9D_Vg) — concise video series
- [Khan Academy Linear Algebra](https://www.khanacademy.org/math/linear-algebra) — free, self-paced
- [Immersive Linear Algebra](http://immersivemath.com/ila/learnmore.html) — interactive visuals
- [3Blue1Brown: Essence of Linear Algebra](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) — outstanding geometric intuition

### 3. Calculus (Gradients)

Neural networks train by gradient descent. You need to understand what a gradient is and how the chain rule enables backpropagation. You do not need to derive backpropagation by hand — understanding the intuition is enough.

Key topics: derivatives, partial derivatives, the chain rule, gradient descent.

- [3Blue1Brown: Calculus series](https://www.youtube.com/playlist?list=PLZHQObOWTQDMsr9K-rj53DwVRMYO3t5Yr)
- [Calculus Made Easy (free book)](http://www.gutenberg.org/ebooks/33283)

### 4. Machine Learning Concepts

Before deep learning, it helps to understand what machine learning is solving: learning a function from data by minimizing a loss. Key concepts:

- Supervised learning (classification, regression)
- Overfitting and generalization
- Train/validation/test splits
- Evaluation metrics (accuracy, F1, RMSE)
- Regularization (L1, L2, dropout)

- [Andrew Ng's Machine Learning Course (YouTube)](https://www.youtube.com/watch?v=qeHZOdmJvFU) — the standard introduction
- [ML@Berkeley: Machine Learning Crash Course Part 1](https://ml.berkeley.edu/blog/2016/11/06/tutorial-1/) and [Part 2](https://ml.berkeley.edu/blog/2016/12/24/tutorial-2/)

---

## The DL4J Ecosystem

DL4J is not just a neural network library. It is a suite of libraries designed to work together:

| Library | Purpose |
|---|---|
| **ND4J** | N-dimensional array computation (the "NumPy of the JVM"); provides INDArray and GPU/CPU ops |
| **DataVec** | Data ingestion, transformation, and vectorization pipeline |
| **Deeplearning4j** | Neural network layers, training, model serialization |
| **SameDiff** | Automatic differentiation (TensorFlow-style define-and-run computation graphs) |
| **Arbiter** | Hyperparameter optimization (grid search, random search) |
| **DL4J Spark** | Distributed training on Apache Spark |
| **Model Zoo** | Pretrained model architectures and weights |

For most applications, you will interact primarily with DL4J and DataVec. ND4J is the foundation that everything runs on, but you only manipulate it directly when working with raw arrays.

---

## Recommended Learning Path

1. **Install Java and IntelliJ** — Use JDK 11 or later. IntelliJ Community Edition is free and has excellent Maven integration.

2. **Clone the examples repository** — The fastest way to see working code:
   ```
   git clone https://github.com/eclipse/deeplearning4j-examples.git
   ```
   Then follow the [Quickstart](quickstart.md) to build and run your first example.

3. **Run the MNIST example** — `MLPMnistSingleLayerExample.java` is the canonical "hello world" for DL4J. It loads data, configures a network, trains it, and evaluates accuracy — all in one file.

4. **Understand the training loop** — Read the [Neural Network Configuration](nn/index.md) guide to understand what each builder call in a `MultiLayerConfiguration` does.

5. **Experiment with the Training UI** — Add the [Training UI](ui/overview.md) to any example and watch the loss curve and parameter statistics in real time. This is the fastest way to develop intuition for hyperparameter tuning.

6. **Try a CNN on your own images** — The `AnimalsClassification.java` example shows how to load images from a directory and train a convolutional network. Swap in your own dataset.

7. **Try transfer learning** — The [Model Zoo](zoo/overview.md) provides pretrained ImageNet weights. Loading VGG16 and replacing the output layer for a 10-class problem is a single-page exercise.

---

## Key Concepts to Understand Early

**INDArray** — DL4J's multidimensional array type. Most bugs for beginners come from array shape mismatches. Learn to read `.shape()` output and understand NCHW vs. NHWC ordering for images.

**DataSetIterator** — The interface for feeding data to a network. DL4J has iterators for MNIST, CSV files, image directories, sequence data, and more. Understand that `hasNext()` + `next()` gives you one minibatch.

**MultiLayerNetwork vs ComputationGraph** — `MultiLayerNetwork` is for simple feedforward and recurrent nets (layer 1 -> layer 2 -> ... -> output). `ComputationGraph` is for networks with multiple inputs, multiple outputs, or skip connections (ResNet, Inception, etc.). When in doubt, prefer `MultiLayerNetwork` to start.

**Updaters vs Optimizers** — In DL4J, the "updater" (Adam, SGD, RMSProp, etc.) is set per layer inside the network configuration, not as a separate optimizer object. This is different from PyTorch/Keras.

**Listeners** — Lightweight callbacks that run after each iteration. `ScoreIterationListener(10)` prints the loss every 10 iterations. `StatsListener` feeds the Training UI. You can write your own.

---

## Free Online Courses

- [Andrew Ng's Deep Learning Specialization (Coursera)](https://www.coursera.org/specializations/deep-learning) — the most complete structured course; can be audited for free
- [Geoff Hinton's Neural Networks on YouTube](https://youtu.be/2fRnHVVLf1Y)
- [Andrej Karpathy's CS231n: CNNs for Visual Recognition](http://cs231n.github.io) — best treatment of convolutional networks; all materials are publicly available
- [Fast.ai Practical Deep Learning](https://course.fast.ai) — top-down, practical approach; uses Python but the concepts transfer directly

---

## Recommended External Reading

- **Deep Learning** (Goodfellow, Bengio, Courville) — the standard textbook; [free online](https://www.deeplearningbook.org/)
- [The Matrix Calculus You Need for Deep Learning](https://explained.ai/matrix-calculus/) — concise, practical
- [Seeing Theory: Visual Introduction to Probability and Statistics](http://students.brown.edu/seeing-theory/)
- [Probability Cheatsheet](https://static1.squarespace.com/static/54bf3241e4b0f0d81bf7ff36/t/55e9494fe4b011aed10e48e5/1441352015658/probability_cheatsheet.pdf)

---

## Getting Help

- **Gitter:** [gitter.im/deeplearning4j/deeplearning4j](https://gitter.im/deeplearning4j/deeplearning4j)
- **GitHub Issues:** [github.com/eclipse/deeplearning4j/issues](https://github.com/eclipse/deeplearning4j/issues)
- **Stack Overflow:** Tag your question with `deeplearning4j`
- **Community Forum:** [community.konduit.ai](https://community.konduit.ai)

When asking for help, include: your DL4J version, the full stack trace, and a minimal reproducible example. Most questions are answered faster when the code can be run directly.
