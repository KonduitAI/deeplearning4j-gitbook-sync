---
title: "Updaters"
description: "Optimization algorithms in ND4J — Adam, SGD, AdaGrad, learning rate schedules, and per-layer updater configuration"
---

# Updaters (Optimizers)

Updaters control how gradients are used to update network parameters during training. They implement different optimization algorithms that adapt learning rates, accumulate momentum, or both. All updater classes are in `org.nd4j.linalg.learning.config`.

## Usage

Pass an updater instance to the network configuration builder:

```java
import org.nd4j.linalg.learning.config.Adam;

new NeuralNetConfiguration.Builder()
    .updater(new Adam(1e-3))       // learning rate = 0.001
    // ... layers ...
    .build();
```

> **Migration note (beta4 to M2.1):** The enum-based form `.updater(Updater.ADAM)` and `.learningRate(0.001)` are removed. Pass the learning rate directly to the updater constructor: `new Adam(1e-3)`.

## Available Updaters

### Adam

```java
new Adam(learningRate)
new Adam(learningRate, beta1, beta2, epsilon)
```

The default and recommended updater for most tasks. Combines momentum (exponential moving average of gradients) with adaptive per-parameter learning rates (exponential moving average of squared gradients).

**Parameters:**
- `learningRate`: Step size (default: 1e-3)
- `beta1`: Exponential decay rate for first moment estimates (default: 0.9)
- `beta2`: Exponential decay rate for second moment estimates (default: 0.999)
- `epsilon`: Small constant for numerical stability (default: 1e-8)

**Update rule:**
```
m_t = beta1 * m_{t-1} + (1 - beta1) * g_t
v_t = beta2 * v_{t-1} + (1 - beta2) * g_t^2
m_hat = m_t / (1 - beta1^t)
v_hat = v_t / (1 - beta2^t)
theta = theta - lr * m_hat / (sqrt(v_hat) + epsilon)
```

### AMSGrad

```java
new AMSGrad(learningRate)
new AMSGrad(learningRate, beta1, beta2, epsilon)
```

A variant of Adam with guaranteed convergence. Uses the maximum of all past squared gradient moving averages instead of the current one, preventing the effective learning rate from increasing.

**Reference:** *On the Convergence of Adam and Beyond* — Reddi et al., 2018

### AdaBelief

```java
new AdaBelief(learningRate)
new AdaBelief(learningRate, beta1, beta2, epsilon)
```

Adapts the learning rate based on the "belief" in the gradient — how much the observed gradient deviates from the predicted gradient. Combines the fast convergence of Adam with the stability of SGD.

**Reference:** *AdaBelief Optimizer: Adapting Stepsizes by the Belief in Observed Gradients* — Zhuang et al., 2020

### Nadam

```java
new Nadam(learningRate)
new Nadam(learningRate, beta1, beta2, epsilon)
```

Adam with Nesterov momentum. Uses the look-ahead gradient (Nesterov momentum) in the update step instead of the current gradient. Often converges faster than standard Adam.

**Reference:** *Incorporating Nesterov Momentum into Adam* — Dozat, 2016

### AdaMax

```java
new AdaMax(learningRate)
new AdaMax(learningRate, beta1, beta2, epsilon)
```

A variant of Adam based on the infinity norm. More robust to large gradients than standard Adam. Uses the max of the exponentially weighted infinity norm of past gradients.

**Reference:** *Adam: A Method for Stochastic Optimization* — Kingma & Ba, 2014 (Section 7)

### AdaGrad

```java
new AdaGrad(learningRate)
new AdaGrad(learningRate, epsilon)
```

Adapts the learning rate per parameter based on the history of gradients. Parameters with large historical gradients get smaller learning rates, and vice versa. Useful for sparse features (NLP, recommender systems).

**Downside:** Learning rate monotonically decreases and can become too small, effectively stopping training.

### AdaDelta

```java
new AdaDelta()
new AdaDelta(rho, epsilon)
```

An extension of AdaGrad that addresses the monotonically decreasing learning rate. Uses a running window of gradient updates instead of accumulating all past gradients. Does not require a learning rate parameter.

**Parameters:**
- `rho`: Decay rate for the running average (default: 0.95)
- `epsilon`: Numerical stability constant (default: 1e-6)

### Nesterovs (SGD with Nesterov Momentum)

```java
new Nesterovs(learningRate)
new Nesterovs(learningRate, momentum)
```

Stochastic gradient descent with Nesterov's accelerated gradient. Evaluates the gradient at the "look-ahead" position rather than the current position, leading to faster convergence than standard momentum.

**Parameters:**
- `learningRate`: Step size
- `momentum`: Momentum coefficient (default: 0.9)

### RmsProp

```java
new RmsProp(learningRate)
new RmsProp(learningRate, rmsDecay, epsilon)
```

Divides the learning rate by an exponentially decaying average of squared gradients. Effective for recurrent neural networks and non-stationary objectives.

**Parameters:**
- `learningRate`: Step size (default: 1e-3)
- `rmsDecay`: Decay rate for moving average (default: 0.95)
- `epsilon`: Numerical stability constant (default: 1e-8)

### SGD

```java
new Sgd(learningRate)
```

Basic stochastic gradient descent with a fixed learning rate. No momentum, no adaptive rates. Simple but often requires more careful tuning of the learning rate and benefits from learning rate schedules.

**Update rule:** `theta = theta - lr * gradient`

### NoOp

```java
new NoOp()
```

No-operation updater — gradients are computed but parameters are not updated. Use this to freeze specific layers:

```java
new DenseLayer.Builder()
    .nIn(256).nOut(128)
    .updater(new NoOp())    // this layer's weights will not change
    .build()
```

## Updater Comparison

| Updater | Learning Rate Required | Adaptive | Momentum | Best For |
|---------|----------------------|----------|----------|----------|
| Adam | Yes | Yes | Yes | Default choice — works well for most tasks |
| AMSGrad | Yes | Yes | Yes | When Adam doesn't converge |
| AdaBelief | Yes | Yes | Yes | When Adam is unstable |
| Nadam | Yes | Yes | Yes (Nesterov) | Faster convergence than Adam |
| AdaMax | Yes | Yes | Yes | Large/sparse gradients |
| AdaGrad | Yes | Yes | No | Sparse features (NLP) |
| AdaDelta | No | Yes | No | When learning rate tuning is difficult |
| Nesterovs | Yes | No | Yes (Nesterov) | Classic CNN training |
| RmsProp | Yes | Yes | No | RNNs, non-stationary objectives |
| SGD | Yes | No | No | Simple tasks, fine-tuning with small LR |
| NoOp | No | N/A | N/A | Freezing layers |

## Learning Rate Schedules

Instead of a fixed learning rate, pass a schedule to any updater. All schedules implement `ISchedule` and are in `org.nd4j.linalg.schedule`.

```java
import org.nd4j.linalg.schedule.*;

ISchedule schedule = new ExponentialSchedule(ScheduleType.EPOCH, 1e-3, 0.95);
.updater(new Adam(schedule))
```

### Available Schedules

#### ExponentialSchedule

```java
new ExponentialSchedule(ScheduleType.EPOCH, initialRate, gamma)
```

Multiplies the learning rate by `gamma` every epoch (or iteration).

`lr = initialRate * gamma^(epoch)`

```java
// Start at 1e-3, multiply by 0.95 each epoch
ISchedule expSchedule = new ExponentialSchedule(ScheduleType.EPOCH, 1e-3, 0.95);
```

#### StepSchedule

```java
new StepSchedule(ScheduleType.EPOCH, initialRate, decayRate, step)
```

Reduces the learning rate by `decayRate` every `step` epochs.

`lr = initialRate * decayRate^(floor(epoch/step))`

```java
// Halve every 10 epochs
ISchedule stepSchedule = new StepSchedule(ScheduleType.EPOCH, 1e-3, 0.5, 10);
```

#### PolySchedule

```java
new PolySchedule(ScheduleType.ITERATION, initialRate, power, maxIter)
```

Polynomial decay from initial rate to zero over `maxIter` iterations.

`lr = initialRate * (1 - iter/maxIter)^power`

```java
ISchedule polySchedule = new PolySchedule(ScheduleType.ITERATION, 1e-3, 2, 50000);
```

#### SigmoidSchedule

```java
new SigmoidSchedule(ScheduleType.EPOCH, initialRate, decayRate, step)
```

Sigmoid-based decay, providing a smooth transition.

#### InverseSchedule

```java
new InverseSchedule(ScheduleType.ITERATION, initialRate, gamma, power)
```

`lr = initialRate * (1 + gamma * iter)^(-power)`

#### CycleSchedule

```java
new CycleSchedule(ScheduleType.ITERATION, minRate, maxRate, cycleLength)
```

Cyclic learning rate that oscillates between `minRate` and `maxRate`. Useful for escaping local minima.

```java
// Cycle between 1e-4 and 1e-2 every 1000 iterations
ISchedule cycleSchedule = new CycleSchedule(ScheduleType.ITERATION, 1e-4, 1e-2, 1000);
```

#### MapSchedule

```java
new MapSchedule(ScheduleType.EPOCH, lrMap)
```

Manually specify the learning rate at specific epochs or iterations:

```java
Map<Integer, Double> lrMap = new HashMap<>();
lrMap.put(0, 1e-3);     // epochs 0-4: lr = 1e-3
lrMap.put(5, 5e-4);     // epochs 5-9: lr = 5e-4
lrMap.put(10, 1e-4);    // epochs 10+: lr = 1e-4
ISchedule mapSchedule = new MapSchedule(ScheduleType.EPOCH, lrMap);
```

#### FixedSchedule

```java
new FixedSchedule(rate)
```

Constant learning rate — equivalent to passing a double directly. Useful when you need an `ISchedule` type but want a fixed rate.

### ScheduleType

The `ScheduleType` enum controls whether the schedule advances per epoch or per iteration (mini-batch):

| Value | Advances Every | Use When |
|-------|---------------|----------|
| `EPOCH` | End of each epoch | Most common. Rate changes once per full pass through data |
| `ITERATION` | Each mini-batch | Fine-grained control. Useful for cyclic/warm-up schedules |

## Per-Layer Updater Configuration

Different layers can use different updaters or learning rates:

```java
new NeuralNetConfiguration.Builder()
    .updater(new Adam(1e-3))            // default for all layers
    .list()
    .layer(new DenseLayer.Builder()
        .nIn(784).nOut(256)
        .activation(Activation.RELU)
        .build())                        // uses Adam(1e-3)
    .layer(new DenseLayer.Builder()
        .nIn(256).nOut(128)
        .updater(new Sgd(0.01))          // override: this layer uses SGD
        .activation(Activation.RELU)
        .build())
    .layer(new OutputLayer.Builder(new LossMCXENT())
        .nIn(128).nOut(10)
        .updater(new Adam(1e-4))         // override: smaller LR for output
        .activation(Activation.SOFTMAX)
        .build())
    .build();
```

This is particularly useful for transfer learning, where pretrained layers might use a smaller learning rate while new layers train with a larger one.

## Practical Recommendations

1. **Start with Adam(1e-3)** — it works well out of the box for most tasks.
2. **If Adam isn't converging**, try AMSGrad or reduce the learning rate.
3. **For fine-tuning pretrained models**, use SGD with a small learning rate (1e-4 to 1e-5) and momentum.
4. **For RNNs**, Adam or RmsProp tend to work better than SGD.
5. **Use learning rate schedules** when training for many epochs — a fixed rate that's good early on may be too large later.
6. **Use per-layer updaters** for transfer learning — freeze early layers with `NoOp` and train later layers with Adam.
