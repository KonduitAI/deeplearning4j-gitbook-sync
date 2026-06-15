---
title: "SameDiff Variables"
description: "SDVariable types — VARIABLE, CONSTANT, PLACEHOLDER, ARRAY — data types, naming, and type conversion"
---

# Variables in SameDiff

## What Variables Are

Every value that flows through a `SameDiff` graph — weights, biases, inputs, labels, intermediate activations — is represented by an object of class `SDVariable`. Unlike a plain `INDArray`, an `SDVariable` is a **node in the computation graph**: it knows its place in the graph, how it was produced, and (when it holds trainable parameters) how to receive gradient updates.

Variables in SameDiff are typically multidimensional arrays, not scalars. When you define `SDVariable w = sd.var("w", DataType.FLOAT, 784, 256)`, you are declaring a 784×256 matrix of floats as a graph node — not a single number.

## Variable Types

All variables in a `SameDiff` instance belong to exactly one of four **variable types**, defined by the `VariableType` enum.

### VARIABLE

`VARIABLE` variables are the **trainable parameters** of your model — things like layer weights and biases.

Properties:
- **Persistent**: their values are stored inside the `SameDiff` instance between calls.
- **Trainable**: they receive gradient updates during `fit()`.
- **Floating-point only**: because gradient descent only makes sense for real-valued parameters.
- **Gradient computation**: gradients with respect to these variables are computed during the backward pass.

Create a `VARIABLE` with `sd.var(...)`:

```java
// Zero-initialised 784x256 weight matrix
SDVariable weights = sd.var("weights", DataType.FLOAT, 784, 256);
```

You will almost always want non-zero initialisation. Pass an `INDArray` directly:

```java
SDVariable weights = sd.var("weights", Nd4j.randn(DataType.FLOAT, 784, 256).div(28.0));
```

Or use one of the built-in weight initialisation schemes:

```java
// Xavier / Glorot initialisation
SDVariable weights = sd.var("weights",
    new XavierInitScheme('c', 784, 256),
    DataType.FLOAT,
    784, 256);
```

Other available schemes include `UniformInitScheme`, `NormalInitScheme`, `ConstantInitScheme`, `ZeroInitScheme`, and more — see the `WeightInitScheme` hierarchy in the [javadoc](https://deeplearning4j.org/api/latest/org/nd4j/weightinit/WeightInitScheme.html).

### CONSTANT

`CONSTANT` variables hold values that are **stored but not updated during training**. They are useful for:

- Hyperparameters you want to bake into the graph (e.g. a dropout rate or a temperature scalar).
- Pre-trained weights from another model that you want to keep frozen.
- Lookup tables or fixed embeddings.

Properties:
- **Persistent**: stored in the `SameDiff` instance.
- **Not trainable**: never receive gradient updates.
- **Any data type**: integers, booleans, and floats are all allowed.
- **No gradient computation**: gradients are not computed for constants.

Create a `CONSTANT` from an `INDArray`:

```java
SDVariable pi = sd.constant("pi", Nd4j.scalar(3.14159f));
SDVariable fixedWeights = sd.constant("frozen_w", pretrainedArray);
```

For a constant holding a single scalar value, use the convenience `scalar` method:

```java
SDVariable temperature = sd.scalar("temperature", 0.07f);
```

Constants can hold any data type:

```java
SDVariable numClasses = sd.constant("num_classes", Nd4j.scalar(DataType.INT, 10));
```

### PLACEHOLDER

`PLACEHOLDER` variables are **slots that receive external data at runtime**. They are the entry points for per-batch data: inputs, labels, masks, etc.

Properties:
- **Not persistent**: their values are not stored between calls. You must supply a value every time you execute the graph.
- **Not trainable**: no gradient updates.
- **Any data type**: depends on what your data looks like.
- **No gradient computation**: by default gradients are not propagated through placeholders (though `ARRAY`-type results downstream do have gradients computed for use in weight updates).

Create a placeholder with `sd.placeHolder(...)`:

```java
// Batch of MNIST images: batch size unknown at graph-definition time (-1), 784 features
SDVariable input = sd.placeHolder("input", DataType.FLOAT, -1, 784);

// Corresponding one-hot labels
SDVariable labels = sd.placeHolder("labels", DataType.FLOAT, -1, 10);
```

The `-1` convention for batch dimension tells SameDiff that the size is unknown at graph-definition time and will be inferred from the actual data at execution time.

You can supply placeholder values explicitly for direct execution:

```java
Map<String, INDArray> placeholderValues = new HashMap<>();
placeholderValues.put("input", myInputArray);
placeholderValues.put("labels", myLabelArray);

Map<String, INDArray> out = sd.output(placeholderValues, "softmax");
```

### ARRAY

`ARRAY` variables are the **intermediate and output values produced by operations** within the graph. Every time you call an op — `.add()`, `.mmul()`, `sd.nn.relu()`, etc. — the result is an `ARRAY`-type variable.

Properties:
- **Not persistent**: recomputed from scratch on every forward pass.
- **Not trainable**: not directly updated.
- **Any data type**: determined by the operation and its inputs.
- **Gradient computation**: gradients *are* computed through `ARRAY` nodes so the chain rule can propagate back to `VARIABLE` nodes.

`ARRAY` variables are created implicitly by operations:

```java
SDVariable z = x.add(y);     // z is ARRAY type
SDVariable h = sd.nn.relu("hidden", z);   // h is also ARRAY type
```

You do not create `ARRAY` variables directly — they are produced by ops.

## Inspecting Variable Type

To find out the type of any variable at runtime:

```java
VariableType vt = myVar.getVariableType();
// Returns VariableType.VARIABLE, CONSTANT, PLACEHOLDER, or ARRAY
```

## Data Types

Each `SDVariable` also has a **data type** (`DataType` enum) that describes the element type of its underlying array.

| Category | Types |
|---|---|
| Floating point | `FLOAT`, `DOUBLE`, `HALF`, `BFLOAT16` |
| Signed integer | `LONG`, `INT`, `SHORT`, `BYTE` |
| Unsigned integer | `UINT64`, `UINT32`, `UINT16`, `UBYTE` |
| Boolean | `BOOL` |
| String | `UTF8` |
| Internal | `COMPRESSED`, `UNKNOWN` |

Retrieve the data type of a variable:

```java
DataType dt = myVar.dataType();
```

Data type matters for operations. For example, `sd.cnn.conv2d(input, weights, config)` requires both `input` and `weights` to be floating-point; passing integer arrays throws an exception. All `VARIABLE`-type variables must be floating-point because gradient descent requires real arithmetic.

When you need to change the data type of an array before passing it to an operation, use `sd.math.castTo(variable, DataType.FLOAT)`.

## Recap Table

The following table summarises all four variable types and their key properties:

| | Trainable | Gradients | Persistent | Workspaces | Data types | Created by |
|---|---|---|---|---|---|---|
| `VARIABLE` | Yes | Yes | Yes | Yes | Float only | `sd.var()` |
| `CONSTANT` | No | No | Yes | No | Any | `sd.constant()` / `sd.scalar()` |
| `PLACEHOLDER` | No | No | No | No | Any | `sd.placeHolder()` |
| `ARRAY` | No | Yes | No | Yes | Any | Operations |

"Workspaces" is an internal memory-management term describing whether the variable's backing memory can be allocated in off-heap workspace regions. You do not need to manage this manually.

## Changing Variable Types

SameDiff supports converting variables between types in a limited set of combinations. This is most commonly used for transfer learning scenarios.

### Variable to Constant (freezing weights)

Turn a trainable `VARIABLE` into a frozen `CONSTANT`. After conversion, the variable will no longer receive gradient updates:

```java
sd.convertToConstant(trainableLayer);
```

Use this when you want to lock specific layers during fine-tuning.

### Constant to Variable (unfreezing weights)

Convert a `CONSTANT` back into a trainable `VARIABLE`. The constant must be of a floating-point data type (because `VARIABLE` requires floating point):

```java
sd.convertToVariable(frozenWeights);  // now trainable again
```

### Placeholder to Constant (fixing an input)

Convert a `PLACEHOLDER` to a `CONSTANT`. Since placeholders are not persistent, you must set the value before converting:

```java
inputPlaceholder.setArray(fixedArray);
sd.convertToConstant(inputPlaceholder);
```

After this, the "input" is baked into the graph as a constant. This can be useful when you have a model that was originally designed to receive inputs dynamically but you now want to specialise it to a fixed input (e.g. a fixed prompt embedding in an NLP pipeline).

Note: there is currently no API to convert a `CONSTANT` back to a `PLACEHOLDER`. If you need to temporarily fix a placeholder but retain the ability to supply new data later, supply the constant value through the normal placeholder mechanism instead of converting the type.

## Naming Variables

Every `SDVariable` in a `SameDiff` instance has a **unique string name**. SameDiff uses names to track variables internally and lets you retrieve them by name later.

### Explicit naming

Pass the name as the first argument to `sd.var()`, `sd.constant()`, `sd.placeHolder()`, or to any op:

```java
SDVariable w = sd.var("encoder_w1", DataType.FLOAT, 512, 256);
SDVariable h = sd.nn.relu("encoder_h1", linear);
```

### Automatic naming

If you do not supply a name, SameDiff generates one automatically based on the operation. Auto-generated names are unique but less readable:

```java
SDVariable z = x.add(y);  // name assigned automatically, e.g. "add:0"
```

### Retrieving variables by name

```java
SDVariable encoderW = sd.getVariable("encoder_w1");
```

This is particularly useful when the `SameDiff` instance was built in a different method or class, or loaded from disk, and you need to get hold of a specific node without keeping a local Java reference.

### Renaming variables

```java
myVar.setVarName("new_name");  // name must remain unique within the SameDiff
```

You can also read the current name:

```java
String name = myVar.name();   // or myVar.getVarName()
```

## Getting Variable Values

### eval()

For persistent variables (`VARIABLE` and `CONSTANT`), call `eval()` to retrieve the current value as an `INDArray`:

```java
INDArray currentWeights = w1.eval();
```

For non-persistent variables (`PLACEHOLDER` and `ARRAY`), you must ensure a value has been set (by executing the graph) before calling `eval()`.

### getArr()

An alternative to `eval()`:

```java
INDArray arr = myVar.getArr();          // returns null if not yet computed
INDArray arr = myVar.getArr(true);      // throws exception if not initialised
```

### Gradients

For variables that have gradients computed (types `VARIABLE` and `ARRAY`), retrieve the gradient array after a backward pass:

```java
INDArray grad = w1.getGradient().eval();
```

Note that gradient values are only available after a backward pass has been executed (e.g. after at least one step of `fit()`, or after an explicit call to `sd.execBackwards()`).

## All Variables in the Graph

To inspect all variables currently registered in a `SameDiff` instance:

```java
// All variable names
List<String> names = sd.variableNames();

// All SDVariable objects
Collection<SDVariable> vars = sd.variables();

// Just the trainable VARIABLE-type parameters
List<SDVariable> trainable = sd.getTrainableVariables();
```
