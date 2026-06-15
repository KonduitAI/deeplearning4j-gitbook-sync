---
title: Arbiter Parameter Spaces
description: Parameter space types for Arbiter hyperparameter search — continuous, integer, discrete, boolean, fixed, and composite spaces.
---

## Overview

A `ParameterSpace<T>` defines the set of valid values for a single hyperparameter. Arbiter uses parameter spaces to sample candidate configurations during random search, or to enumerate all combinations during grid search.

All parameter spaces are in the `arbiter-core` module and live under `org.deeplearning4j.arbiter.optimize.parameter`.

---

## Primitive Parameter Spaces

### ContinuousParameterSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/continuous/ContinuousParameterSpace.java)

Defines a continuous range of `double` values with a uniform distribution between a minimum and maximum.

```java
// Uniform in [0.0001, 0.01]
ParameterSpace<Double> learningRate = new ContinuousParameterSpace(0.0001, 0.01);
```

**Constructor:**

```java
ContinuousParameterSpace(double min, double max)
```

**Usage in layer builder:**

```java
new DenseLayerSpace.Builder()
    .l2(new ContinuousParameterSpace(1e-6, 1e-3))
    .build();
```

For scale-sensitive hyperparameters like learning rate, consider `LogUniformDistribution` which is more likely to sample values across orders of magnitude evenly. `ContinuousParameterSpace` samples uniformly in linear space, which over-samples the high end.

---

### IntegerParameterSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/integer/IntegerParameterSpace.java)

Defines an integer range with a uniform distribution.

```java
// Uniform integer in [64, 512], inclusive
ParameterSpace<Integer> layerSize = new IntegerParameterSpace(64, 512);
```

**Constructor:**

```java
IntegerParameterSpace(int min, int max)
```

Min and max are both inclusive.

```java
// Number of layers: 1, 2, 3, or 4
ParameterSpace<Integer> numLayers = new IntegerParameterSpace(1, 4);
```

**Accessor methods:**

```java
int min = space.getMin();   // 64
int max = space.getMax();   // 512
```

---

### DiscreteParameterSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/discrete/DiscreteParameterSpace.java)

Defines an unordered set of discrete values. The search strategy samples uniformly from the set.

```java
// Discrete set of activation functions
ParameterSpace<Activation> activation = new DiscreteParameterSpace<>(
    Activation.RELU,
    Activation.TANH,
    Activation.ELU,
    Activation.LEAKYRELU
);
```

**Constructor:**

```java
DiscreteParameterSpace<T>(T... values)
DiscreteParameterSpace<T>(List<T> values)
```

The type parameter `T` can be any Java object, including arrays:

```java
// Kernel sizes: 3x3 or 5x5
ParameterSpace<int[]> kernelSize = new DiscreteParameterSpace<>(
    new int[]{3, 3},
    new int[]{5, 5}
);
```

With custom loss functions:

```java
ILossFunction[] lossFunctions = {
    new LossMCXENT(Nd4j.create(new double[]{1.0, 2.0})),
    new LossMCXENT(Nd4j.create(new double[]{1.0, 5.0})),
    new LossMCXENT(Nd4j.create(new double[]{1.0, 10.0}))
};
ParameterSpace<ILossFunction> lossSpace = new DiscreteParameterSpace<>(lossFunctions);
```

---

### BooleanSpace

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/BooleanSpace.java)

Samples `true` or `false` with equal probability. Internally, the space maps from a continuous value: values <= 0.5 map to `true`, values > 0.5 map to `false`.

```java
ParameterSpace<Boolean> hasBias = new BooleanSpace();
```

**Usage:**

```java
new MultiLayerSpace.Builder()
    .hasBias(new BooleanSpace())
    // ...
```

---

### FixedValue

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/FixedValue.java)

A `ParameterSpace` that always returns the same value. Useful for mixing fixed and variable hyperparameters in the same builder, or for locking a parameter that you want to exclude from the search:

```java
// Fix the learning rate to exactly 0.001 across all candidates
ParameterSpace<Double> fixedLR = new FixedValue<>(0.001);

// Fix the number of added layers to 3
ParameterSpace<Integer> fixedLayers = new FixedValue<>(3);
```

**Constructor:**

```java
FixedValue<T>(T value)
```

---

## Composite Parameter Spaces

### MathOp

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/math/MathOp.java)

Applies a scalar mathematical operation to another parameter space. This allows you to derive one hyperparameter as a function of another.

```java
ParameterSpace<Integer> firstLayerSize = new IntegerParameterSpace(64, 256);

// Second layer is 2x the first layer's size
ParameterSpace<Integer> secondLayerSize = new MathOp<>(
    firstLayerSize,
    Op.MUL,
    2
);
```

Available operations (`MathOp.Op`):

| Op | Description |
|---|---|
| `ADD` | `result = space + scalar` |
| `SUB` | `result = space - scalar` |
| `MUL` | `result = space * scalar` |
| `DIV` | `result = space / scalar` |
| `MOD` | `result = space % scalar` |

The scalar is always on the right-hand side. The space value and the scalar must have compatible types.

**Full example:**

```java
ParameterSpace<Double> baseLR = new ContinuousParameterSpace(1e-4, 1e-2);

// Use 10x smaller LR for fine-tuning layers
ParameterSpace<Double> fineTuneLR = new MathOp<>(baseLR, Op.DIV, 10.0);

MultiLayerSpace mls = new MultiLayerSpace.Builder()
    .addLayer(new DenseLayerSpace.Builder()
        .updater(new AdamSpace(baseLR))
        .nOut(256)
        .build())
    .addLayer(new OutputLayerSpace.Builder()
        .updater(new AdamSpace(fineTuneLR))  // correlated with baseLR
        .nOut(10)
        .build())
    .numEpochs(20)
    .build();
```

When a candidate is generated, `firstLayerSize` is sampled once and `secondLayerSize` is derived from it deterministically. All candidates will satisfy the relationship `secondLayerSize = 2 * firstLayerSize`.

---

### PairMathOp

[[source]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/math/PairMathOp.java)

Applies a mathematical operation to two independent parameter spaces. This allows you to create a derived parameter from two sampled values.

```java
ParameterSpace<Double> l1Space = new ContinuousParameterSpace(1e-6, 1e-4);
ParameterSpace<Double> l2Space = new ContinuousParameterSpace(1e-6, 1e-4);

// Total regularization = l1 + l2
ParameterSpace<Double> totalReg = new PairMathOp<>(l1Space, Op.ADD, l2Space);
```

Available operations (`PairMathOp.Op`): `ADD`, `SUB`, `MUL`, `DIV`, `MOD`.

---

## Updater Spaces

Arbiter provides updater-specific spaces in `org.deeplearning4j.arbiter.optimize.parameter.updater` (and `org.deeplearning4j.arbiter.conf.updater`). These allow the optimizer itself to be part of the hyperparameter search.

### AdamSpace

```java
ParameterSpace<IUpdater> updaterSpace = new AdamSpace(
    new ContinuousParameterSpace(1e-4, 1e-2)  // learning rate
);
```

### SgdSpace

```java
ParameterSpace<IUpdater> updaterSpace = new SgdSpace(
    new ContinuousParameterSpace(1e-3, 0.1)
);
```

### NesterovSpace

```java
ParameterSpace<IUpdater> updaterSpace = new NesterovSpace(
    new ContinuousParameterSpace(1e-3, 0.1)
);
```

### RmsPropSpace

```java
ParameterSpace<IUpdater> updaterSpace = new RmsPropSpace(
    new ContinuousParameterSpace(1e-4, 1e-2)
);
```

Use `DiscreteParameterSpace` to include the updater choice itself in the search:

```java
ParameterSpace<IUpdater> updaterSpace = new DiscreteParameterSpace<>(
    new Adam(0.001),
    new Nesterovs(0.01, 0.9),
    new RmsProp(0.001)
);
```

---

## How Arbiter Samples Parameter Spaces

All parameter spaces ultimately map a `double[]` input (sampled uniformly from `[0, 1]`) to a concrete value. The mapping varies by space type:

| Space | Mapping |
|---|---|
| `ContinuousParameterSpace` | `min + input[0] * (max - min)` |
| `IntegerParameterSpace` | `(int)(min + input[0] * (max - min + 1))` clamped to `[min, max]` |
| `DiscreteParameterSpace` | `values[(int)(input[0] * values.length)]` |
| `BooleanSpace` | `input[0] <= 0.5` |
| `FixedValue` | always returns the stored value |
| `MathOp` | samples the inner space, then applies the operation |

For random search, the `double[]` inputs are drawn uniformly. For grid search, they are enumerated at the specified `discretizationCount` steps.

---

## Implementing a Custom ParameterSpace

You can implement `ParameterSpace<T>` to support domain-specific sampling logic:

```java
public class PowerOfTwoSpace implements ParameterSpace<Integer> {
    private final int minExp;
    private final int maxExp;

    public PowerOfTwoSpace(int minExp, int maxExp) {
        this.minExp = minExp;
        this.maxExp = maxExp;
    }

    @Override
    public Integer getValue(double[] parameterValues) {
        int exp = minExp + (int)(parameterValues[0] * (maxExp - minExp + 1));
        return 1 << Math.min(exp, maxExp);
    }

    @Override
    public int numParameters() { return 1; }

    @Override
    public List<ParameterSpace> collectLeaves() {
        return Collections.singletonList(this);
    }

    @Override
    public boolean isLeaf() { return true; }

    @Override
    public void setIndices(int... indices) { /* store index */ }
}
```

Usage:

```java
// Layer size sampled from {32, 64, 128, 256, 512}
ParameterSpace<Integer> powerOfTwoSize = new PowerOfTwoSpace(5, 9);
```

---

## Related Pages

- [Arbiter Overview](./overview.md) — optimization configuration and runner
- [Layer Spaces](./layer-spaces.md) — per-layer hyperparameter spaces
- [Visualization](./visualization.md) — monitoring the optimization run
