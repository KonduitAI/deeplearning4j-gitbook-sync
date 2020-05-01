---
title: Arbiter Parameter Spaces
short_title: Parameter Spaces
description: Set a search spaces for parameters.
category: Arbiter
weight: 1
---

# Parameter Spaces

### BooleanSpace

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/BooleanSpace.java)

If argument to setValue is less than or equal to 0.5 it will return True else False

### FixedValue

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/FixedValue.java)

FixedValue is a ParameterSpace that defines only a single fixed value

### ContinuousParameterSpace

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/continuous/ContinuousParameterSpace.java)

**getValue**

```text
public Double getValue(double[] input) 
```

ContinuousParameterSpace with uniform distribution between the minimum and maximum values

* param min Minimum value that can be generated
* param max Maximum value that can be generated

### DiscreteParameterSpace

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/discrete/DiscreteParameterSpace.java)

A DiscreteParameterSpace is used for a set of un-ordered values

### IntegerParameterSpace

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/integer/IntegerParameterSpace.java)

some minimum and maximum value

#### **getMin**

```text
public int getMin() 
```

Create an IntegerParameterSpace with a uniform distribution between the specified min/max \(inclusive\)

* param min Min value, inclusive
* param max Max value, inclusive

### MathOp

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/math/MathOp.java)

A simple parameter space that implements scalar mathematical operations on another parameter space. This allows you to do things like Y = X 2, where X is a parameter space. For example, a layer size hyperparameter could be set using this to 2x the size of the previous layer

### PairMathOp

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/arbiter/arbiter-core/src/main/java/org/deeplearning4j/arbiter/optimize/parameter/math/PairMathOp.java)

A simple parameter space that implements pairwise mathematical operations on another parameter space. This allows you to do things like Z = X + Y, where X and Y are parameter spaces.

