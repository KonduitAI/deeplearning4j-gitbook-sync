---
title: "SameDiff Operations"
description: "Op namespaces — sd.math, sd.nn, sd.cnn, sd.rnn, sd.loss, sd.random — and SDVariable methods"
---

# SameDiff Operations

Operations in SameDiff consume one or more `SDVariable` inputs and produce a new `SDVariable` output of type `ARRAY`. They are the edges of the computation graph that connect variable nodes together. The total number of available operations, including overloads, runs into the hundreds — from simple elementwise addition to full LSTM layers.

This page gives an overview of where to find operations, how to use them, and what rules to keep in mind.

## Common Properties

Before looking at individual namespaces, there are a few properties that apply to **all** SameDiff operations:

- **Any variable type is valid as input**, as long as the data types match what the operation requires. Most numeric operations require floating-point inputs.
- **All operation outputs are `ARRAY`-type variables**. Operations never return `VARIABLE`, `CONSTANT`, or `PLACEHOLDER`.
- **Variables used in a single operation must all belong to the same `SameDiff` instance.** Mixing variables from different `SameDiff` objects in one operation is an error.
- **You may optionally name the output variable.** Pass the desired name as the first `String` argument:

```java
SDVariable linear = weights.mmul("matrix_product", input).add(bias);
SDVariable output = sd.nn.sigmoid("output", linear);
```

Named outputs can be retrieved from the graph later with `sd.getVariable(String name)`. If no name is supplied, a unique one is auto-generated from the operation name (e.g. `"mmul:0"`).

## Two Families of Operations

Operations live in two places:

1. **`SDVariable` instance methods** — called directly on a variable, e.g. `x.add(y)`.
2. **`SameDiff` namespace methods** — called via one of six namespace objects on the `SameDiff` instance.

### SDVariable Instance Methods

`SDVariable` exposes a rich set of methods for common operations. These are the most ergonomic to use because they can be chained:

```java
SDVariable result = weights.mmul(input).add(bias);
```

#### Linear algebra (BLAS-style)

| Method | Description |
|---|---|
| `add(y)` | Elementwise addition `x + y` (broadcasting supported) |
| `sub(y)` | Elementwise subtraction `x - y` |
| `mul(y)` | Elementwise multiplication `x * y` (or scalar scaling) |
| `div(y)` | Elementwise division `x / y` |
| `neg()` | Negate all elements |
| `mmul(y)` | Matrix multiplication |
| `dot(y, dimension)` | Dot product along a dimension |
| `rdiv(y)` | Reverse division `y / x` |
| `rsub(y)` | Reverse subtraction `y - x` |

#### Comparison

| Method | Description |
|---|---|
| `gt(y)` | Greater than (element vs scalar or element vs element) |
| `gte(y)` | Greater than or equal |
| `lt(y)` | Less than |
| `lte(y)` | Less than or equal |
| `eq(y)` | Equal |
| `neq(y)` | Not equal |

#### Reductions

Reductions take an optional `int... dimensions` argument. If omitted, the reduction is over all elements.

| Method | Description |
|---|---|
| `sum(dimensions)` | Sum of elements |
| `mean(dimensions)` | Mean of elements |
| `min(dimensions)` | Minimum value |
| `max(dimensions)` | Maximum value |
| `norm1(dimensions)` | L1 norm |
| `norm2(dimensions)` | L2 norm |
| `prod(dimensions)` | Product of elements |
| `argmax(dimensions)` | Index of maximum element |
| `argmin(dimensions)` | Index of minimum element |
| `squaredDifference(y)` | Elementwise `(x - y)^2` |

#### Shape manipulation

| Method | Description |
|---|---|
| `reshape(long... shape)` | Reshape to specified shape |
| `permute(int... dims)` | Permute dimensions (transpose generalisation) |
| `shape()` | Returns the shape as an integer `SDVariable` |

#### Chaining example

```java
// Compute mean squared error in one chained expression
SDVariable mse = predictions.sub(labels).square().mean();
```

## SameDiff Namespace Operations

The `SameDiff` class provides six namespace objects. Access them as fields or method calls (both styles work):

```java
sd.math.sin(x);    // field-style
sd.math().sin(x);  // method-style
```

The six namespaces are: `math`, `random`, `nn`, `cnn`, `rnn`, and `loss`.

### `sd.math` — General Mathematical Operations

The `math` namespace provides a broad collection of mathematical functions, statistics, and linear-algebra primitives.

#### Power and exponential functions

```java
SDVariable y = sd.math.square(x);        // x^2
SDVariable y = sd.math.cube(x);          // x^3
SDVariable y = sd.math.sqrt(x);          // √x
SDVariable y = sd.math.pow(x, 3.0);      // x^3.0
SDVariable y = sd.math.reciprocal(x);    // 1/x
SDVariable y = sd.math.exp(x);           // e^x
SDVariable y = sd.math.log(x);           // natural log
SDVariable y = sd.math.log1p(x);         // log(1 + x)
```

#### Trigonometric and hyperbolic functions

```java
SDVariable s = sd.math.sin(x);
SDVariable c = sd.math.cos(x);
SDVariable t = sd.math.tan(x);
SDVariable a = sd.math.atan(x);
SDVariable h = sd.math.sinh(x);
SDVariable th = sd.math.tanh(x);
SDVariable ah = sd.math.atanh(x);
```

#### Elementwise miscellaneous

```java
SDVariable a = sd.math.abs(x);
SDVariable s = sd.math.sign(x);
SDVariable r = sd.math.round(x);
SDVariable c = sd.math.ceil(x);
SDVariable f = sd.math.floor(x);
SDVariable cl = sd.math.clipByValue(x, -1.0, 1.0);
SDVariable cn = sd.math.clipByNorm(x, 1.0, new int[]{1}); // clip L2 norm per row
```

#### Reductions

```java
SDVariable m  = sd.math.mean(x, 0);          // mean along dim 0
SDVariable mn = sd.math.min(x, 1);           // min along dim 1
SDVariable am = sd.math.amax(x);             // absolute maximum over all elements
SDVariable le = sd.math.logEntropy(x, 0);    // log-entropy along dim 0
```

#### Distance operations (between two identically-shaped variables)

```java
SDVariable ed = sd.math.euclideanDistance(x, y, 1);
SDVariable md = sd.math.manhattanDistance(x, y, 1);
SDVariable cd = sd.math.cosineDistance(x, y, 1);
SDVariable cs = sd.math.cosineSimilarity(x, y, 1);
```

#### Matrix operations

```java
SDVariable inv  = sd.math.matrixInverse(m);
SDVariable det  = sd.math.matrixDeterminant(m);
SDVariable diag = sd.math.diag(v);           // diagonal matrix from vector
SDVariable tr   = sd.math.trace(m);
SDVariable eye  = sd.math.eye(5);            // 5x5 identity matrix
```

#### Logical operations

```java
SDVariable andResult = sd.math.and(a, b);
SDVariable orResult  = sd.math.or(a, b);
SDVariable xorResult = sd.math.xor(a, b);
SDVariable notResult = sd.math.not(a);
```

#### Chaining in `math`

Chaining `math` ops is slightly more verbose than chaining `SDVariable` methods:

```java
// Matrix 1-norm: max column absolute sum
SDVariable norm1 = sd.math.max(sd.math.sum(sd.math.abs(matrix), 0));
```

### `sd.random` — Random Number Generators

The `random` namespace creates variables whose underlying arrays are filled with random values on each forward pass. These are useful for noise injection, dropout masks, or random initialisation inside the graph.

#### Fixed-shape random variables

```java
double mean = 0.0, stddev = 0.05;
SDVariable noise = sd.random.normal("noise", mean, stddev, new long[]{28, 28});

SDVariable uniform = sd.random.uniform("uniform", 0.0, 1.0, new long[]{64, 512});

SDVariable bernoulli = sd.random.bernoulli("mask", 0.5, new long[]{32, 256});

SDVariable binomial = sd.random.binomial("bin", 10, 0.3, new long[]{100});
```

#### Dynamic-shape random variables

When the shape depends on another variable in the graph (e.g. because the batch size is variable), pass an integer `SDVariable` as the shape:

```java
SDVariable windowShape = sd.placeHolder("window_shape", DataType.INT, 2);
SDVariable noise = sd.random.normal("audio_noise", 0.0, 0.02, windowShape);
```

The shape variable must have an integer data type.

### `sd.nn` — Neural Network Layers and Activations

The `nn` namespace covers operations commonly used in general neural networks that are not specific to convolutional or recurrent architectures.

#### Dense layers

```java
// Linear layer: output = input @ weights + bias
SDVariable linear = sd.nn.linear(input, weights, bias);

// ReLU layer in one call
SDVariable reluOut = sd.nn.reluLayer(input, weights, bias);

// Add bias separately
SDVariable withBias = sd.nn.biasAdd(features, bias);
```

#### Activation functions

```java
SDVariable r  = sd.nn.relu("relu", x, 0);         // second arg: leaky coefficient (0 = standard ReLU)
SDVariable lr = sd.nn.leakyRelu(x, 0.01);
SDVariable e  = sd.nn.elu(x);
SDVariable s  = sd.nn.sigmoid(x);
SDVariable t  = sd.nn.tanh(x);
SDVariable ht = sd.nn.hardTanh(x);
SDVariable sm = sd.nn.softmax("sm", x);
SDVariable sp = sd.nn.softplus(x);
SDVariable st = sd.nn.softsign(x);
SDVariable ge = sd.nn.gelu(x);                    // Gaussian Error Linear Unit
SDVariable sw = sd.nn.swish(x);
SDVariable mi = sd.nn.mish(x);
```

#### Regularisation

```java
// Dropout: keep probability 0.8
SDVariable dropped = sd.nn.dropout(x, 0.8);

// Layer normalisation
SDVariable normed = sd.nn.layerNorm("ln", x, gain, bias, false, 1);  // false = not channel-first; norm over dim 1

// Batch normalisation (inference mode)
SDVariable bnOut = sd.nn.batchNorm(x, mean, variance, gamma, beta, 1e-5, 1);
```

#### Padding

```java
// Pad a 2D array symmetrically with zeros, 2 elements on each side
SDVariable padded = sd.nn.pad(x, new int[][]{{2,2},{2,2}}, PadMode.CONSTANT, 0.0);
```

#### Full example: two-layer feedforward network

```java
SDVariable h1 = sd.nn.reluLayer("h1", input, w1, b1);
SDVariable h2 = sd.nn.reluLayer("h2", h1, w2, b2);
SDVariable out = sd.nn.softmax("output", h2.mmul(w3).add(b3));
```

### `sd.cnn` — Convolutional Neural Network Operations

The `cnn` namespace provides convolution, pooling, and related operations.

#### Convolution operations

Convolution layers are specified via **configuration objects** that bundle the many static hyperparameters (kernel size, stride, padding, dilation, data format, bias flag, etc.).

**1D convolution:**

```java
Conv1DConfig cfg1d = Conv1DConfig.builder()
    .k(3)     // kernel width
    .s(1)     // stride
    .p(1)     // padding
    .build();

SDVariable conv1d = sd.cnn.conv1d(input, weights, cfg1d);
// or with bias:
SDVariable conv1d = sd.cnn.conv1d(input, weights, bias, cfg1d);
```

**2D convolution:**

```java
Conv2DConfig cfg2d = Conv2DConfig.builder()
    .kH(3).kW(3)    // kernel height, width
    .pH(1).pW(1)    // padding
    .sH(1).sW(1)    // stride
    .hasBias(false) // no bias (add separately if desired)
    .dataFormat("NCHW")
    .build();

SDVariable conv2d = sd.cnn.conv2d(input, weights, cfg2d);
```

Input shape for `NCHW` format: `[batch, channels_in, height, width]`.
Weight shape: `[channels_out, channels_in, kH, kW]`.

**3D convolution:**

```java
Conv3DConfig cfg3d = Conv3DConfig.builder()
    .kD(3).kH(3).kW(3)
    .build();

SDVariable conv3d = sd.cnn.conv3d(input, weights, cfg3d);
```

**Depthwise and separable convolutions:**

```java
SDVariable dwConv = sd.cnn.depthWiseConv2d(input, depthwiseWeights, cfg2d);
SDVariable sepConv = sd.cnn.separableConv2d(input, depthWeights, pointWeights, cfg2d);
```

#### Deconvolution (transposed convolution)

```java
DeConv2DConfig dcfg = DeConv2DConfig.builder().kH(3).kW(3).build();
SDVariable deconv2d = sd.cnn.deconv2d(input, weights, dcfg);
```

#### Pooling

```java
Pooling2DConfig pool2d = Pooling2DConfig.builder()
    .kH(2).kW(2)
    .sH(2).sW(2)
    .build();

SDVariable maxPool   = sd.cnn.maxPooling2d(x, pool2d);
SDVariable avgPool   = sd.cnn.avgPooling2d(x, pool2d);
SDVariable maxPool1d = sd.cnn.maxPooling1d(x, Pooling1DConfig.builder().k(2).s(2).build());
```

#### Upsampling

```java
SDVariable upsampled = sd.cnn.upsampling2d(x, 2);   // upsample by factor 2
```

#### Local response normalisation

```java
LocalResponseNormalizationConfig lrnCfg = LocalResponseNormalizationConfig.builder()
    .alpha(1e-4).beta(0.75).bias(1.0).depth(5).build();
SDVariable lrn = sd.cnn.localResponseNormalization(x, lrnCfg);
```

#### Full example: simple ConvNet block

```java
// Define config
Conv2DConfig cfg = Conv2DConfig.builder().kH(3).kW(3).pH(1).pW(1).hasBias(true).build();
Pooling2DConfig pool = Pooling2DConfig.builder().kH(2).kW(2).sH(2).sW(2).build();

// Build graph
SDVariable conv   = sd.cnn.conv2d("conv1", input, weights, bias, cfg);
SDVariable act    = sd.nn.relu("act1", conv, 0);
SDVariable pooled = sd.cnn.maxPooling2d("pool1", act, pool);
SDVariable flat   = pooled.reshape(-1, flatSize);
SDVariable logits = sd.nn.linear("logits", flat, fcWeights, fcBias);
SDVariable out    = sd.nn.softmax("output", logits);
```

### `sd.rnn` — Recurrent Neural Network Operations

The `rnn` namespace provides modules for sequence modelling.

#### Simple Recurrent Units (SRU)

```java
SRUConfiguration sruConfig = new SRUConfiguration(input, weights, bias, initialState);
SDVariable sruOutput = sd.rnn.sru(sruConfig);

// Cell-level SRU (single time step)
SRUCellConfiguration sruCellCfg = new SRUCellConfiguration(input, weights, bias, state);
SDVariable[] sruCellOut = sd.rnn.sruCell(sruCellCfg);
```

#### LSTM

```java
LSTMConfiguration lstmCfg = LSTMConfiguration.builder()
    .forgetBias(1.0)
    .clippingCellValue(3.0)
    .build();

// Full LSTM layer (processes all time steps)
SDVariable[] lstmOut = sd.rnn.lstmLayer(input, cLast, yLast, weights, lstmCfg);

// LSTM cell (single time step)
LSTMCellConfiguration cellCfg = LSTMCellConfiguration.builder()
    .x(input)
    .cx(cellState)
    .cs(cLast)
    .h(hLast)
    .wci(wci).wcf(wcf).wco(wco)
    .b(bias)
    .w(weights)
    .build();
SDVariable[] lstmCellOut = sd.rnn.lstmCell(cellCfg);
```

#### GRU

```java
GRUConfiguration gruCfg = GRUConfiguration.builder()
    .x(input)
    .hLast(hLast)
    .wRU(wRU).wC(wC)
    .bRU(bRU).bC(bC)
    .build();

SDVariable gruOutput = sd.rnn.gru(gruCfg);
```

All recurrent outputs are `ARRAY`-type and can be fed into subsequent operations.

### `sd.loss` — Loss Functions

The `loss` namespace provides standard loss functions for training. Most loss functions share a common signature:

```java
SDVariable loss = sd.loss.functionName("loss_name", labels, predictions [, weights, LossReduce]);
```

The `String` name is required (can be `null` for auto-naming). `weights` and `LossReduce` are optional.

#### Common loss functions

```java
// Binary cross-entropy
SDVariable bce = sd.loss.binaryCrossEntropy("bce", labels, predictions, null, LossReduce.MEAN_BY_WEIGHT);

// Softmax cross-entropy (logits, not probabilities)
SDVariable sce = sd.loss.softmaxCrossEntropy("sce", labels, logits, null);

// Log loss
SDVariable ll = sd.loss.logLoss("logLoss", labels, predictions);

// Mean squared error
SDVariable mse = sd.loss.meanSquaredError("mse", labels, predictions, null, LossReduce.MEAN_BY_WEIGHT);

// Mean absolute error
SDVariable mae = sd.loss.absoluteDifference("mae", labels, predictions, null, LossReduce.MEAN_BY_WEIGHT);

// Hinge loss
SDVariable hinge = sd.loss.hingeLoss("hinge", labels, predictions, null, LossReduce.SUM);

// Huber loss (smooth L1)
SDVariable huber = sd.loss.huberLoss("huber", labels, predictions, null, LossReduce.MEAN_BY_WEIGHT, 1.0);

// Cosine distance loss
SDVariable cos = sd.loss.cosineDistance("cosine", labels, predictions, 1, null, LossReduce.MEAN_BY_WEIGHT);
```

#### Reduction methods

The `LossReduce` enum controls how per-sample losses are aggregated over the minibatch:

| `LossReduce` value | Formula | Result shape |
|---|---|---|
| `NONE` | Leave per-sample values as-is | `[batchSize]` |
| `SUM` | `sum(weights * loss_i)` | scalar |
| `MEAN_BY_WEIGHT` | `sum(weights * loss_i) / sum(weights)` | scalar |
| `MEAN_BY_NONZERO_WEIGHT_COUNT` | `sum(weights * loss_i) / count(weights != 0)` | scalar |

When no weights are specified, `MEAN_BY_WEIGHT` and `MEAN_BY_NONZERO_WEIGHT_COUNT` are equivalent to plain mean.

Use `MEAN_BY_NONZERO_WEIGHT_COUNT` when you want to average only over "valid" samples marked with `weight=1`, ignoring padding positions marked with `weight=0`.

#### Weighted loss example

```java
// Per-sample weights: 2x for positive class, 1x for negative
SDVariable classWeights = sd.placeHolder("class_weights", DataType.FLOAT, -1);

SDVariable weightedBce = sd.loss.binaryCrossEntropy(
    "weighted_bce",
    labels, predictions,
    classWeights,
    LossReduce.MEAN_BY_WEIGHT
);
```

## The Don'ts of Operations

A few patterns cause subtle bugs in SameDiff graphs. Avoid them:

### Don't mix variables from different `SameDiff` instances

```java
SameDiff sd0 = SameDiff.create();
SameDiff sd1 = SameDiff.create();

SDVariable x = sd0.var(DataType.FLOAT, 4);
SDVariable y = sd1.placeHolder(DataType.FLOAT, 4);

// BAD: x and y belong to different SameDiff instances
SDVariable z = x.add(y);   // will throw an exception or produce wrong results
```

All variables used in a single op must belong to the same `SameDiff`.

### Don't discard operation results

Every op call creates a new node in the graph. If you call an op without assigning the result to a variable, the node is created but nothing can reference it downstream. This is almost always a bug:

```java
SDVariable z = x.add(y);
z.mul(2);           // BAD: result discarded — this node goes nowhere
x = z.mul(y);      // BAD: reassigning x does not modify the graph node x was pointing to
```

The correct pattern is always to assign the result to a new variable:

```java
SDVariable z   = x.add(y);
SDVariable z2  = z.mul(2);
SDVariable out = z2.mul(y);
```

### Don't redefine existing named variables

If you call `sd.var("weights", ...)` twice with the same name, you will either get an exception or silently reference the same underlying variable. Always use unique names:

```java
// BAD if "w1" already exists in the graph
SDVariable w1 = sd.var("w1", DataType.FLOAT, 128, 64);

// GOOD: distinct names
SDVariable w_encoder = sd.var("encoder_w1", DataType.FLOAT, 128, 64);
SDVariable w_decoder = sd.var("decoder_w1", DataType.FLOAT, 64, 128);
```

## Finding Operations in the Javadoc

The full operation reference is in the [SameDiff javadoc](https://deeplearning4j.org/api/latest/). Navigate to:

- `org.nd4j.autodiff.samediff.SDVariable` for instance methods.
- `org.nd4j.autodiff.samediff.ops.SDMath`, `SDRandom`, `SDNN`, `SDCNN`, `SDRNN`, `SDLoss` for namespace ops.

IDE autocompletion is also effective: type `sd.nn.` and browse the suggestions.
