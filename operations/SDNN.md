---
title: NN
short_title: NN
description: 
category: Operations
weight: 10
---
# NN Namespace
# Operation classes
## <a name="batchNorm">batchNorm</a>
```JAVA
INDArray batchNorm(INDArray input, INDArray mean, INDArray variance, INDArray gamma, INDArray beta, double epsilon, int[] axis)

SDVariable batchNorm(SDVariable input, SDVariable mean, SDVariable variance, SDVariable gamma, SDVariable beta, double epsilon, int[] axis)
SDVariable batchNorm(String name, SDVariable input, SDVariable mean, SDVariable variance, SDVariable gamma, SDVariable beta, double epsilon, int[] axis)
```
Neural network batch normalization operation.

For details, see <a href="https://arxiv.org/abs/1502.03167">https://arxiv.org/abs/1502.03167</a>

* **input** - Input variable. (NUMERIC type)
* **mean** - Mean value. For 1d axis, this should match input.size(axis) (NUMERIC type)
* **variance** - Variance value. For 1d axis, this should match input.size(axis) (NUMERIC type)
* **gamma** - Gamma value. For 1d axis, this should match input.size(axis) (NUMERIC type)
* **beta** - Beta value. For 1d axis, this should match input.size(axis) (NUMERIC type)
* **epsilon** - Epsilon constant for numerical stability (to avoid division by 0)
* **axis** - For 2d CNN activations: 1 for NCHW format activations, or 3 for NHWC format activations.
For 3d CNN activations: 1 for NCDHW format, 4 for NDHWC
For 1d/RNN activations: 1 for NCW format, 2 for NWC (Size: AtLeast(min=1)

## <a name="biasAdd">biasAdd</a>
```JAVA
INDArray biasAdd(INDArray input, INDArray bias, boolean nchw)

SDVariable biasAdd(SDVariable input, SDVariable bias, boolean nchw)
SDVariable biasAdd(String name, SDVariable input, SDVariable bias, boolean nchw)
```
Bias addition operation: a special case of addition, typically used with CNN 4D activations and a 1D bias vector

* **input** - 4d input variable (NUMERIC type)
* **bias** - 1d bias (NUMERIC type)
* **nchw** - The format - nchw=true means [minibatch, channels, height, width] format; nchw=false - [minibatch, height, width, channels].
Unused for 2d inputs

## <a name="dotProductAttention">dotProductAttention</a>
```JAVA
INDArray dotProductAttention(INDArray queries, INDArray keys, INDArray values, INDArray mask, boolean scaled)

SDVariable dotProductAttention(SDVariable queries, SDVariable keys, SDVariable values, SDVariable mask, boolean scaled)
SDVariable dotProductAttention(String name, SDVariable queries, SDVariable keys, SDVariable values, SDVariable mask, boolean scaled)
```
This operation performs dot product attention on the given timeseries input with the given queries

out = sum(similarity(k_i, q) * v_i)



similarity(k, q) = softmax(k * q) where x * q is the dot product of x and q



Optionally with normalization step:

similarity(k, q) = softmax(k * q / sqrt(size(q))



See also "Attention is all you need" (https://arxiv.org/abs/1706.03762, p. 4, eq. 1)



Note: This supports multiple queries at once, if only one query is available the queries vector still has to

be 3D but can have queryCount = 1



Note: keys and values usually is the same array. If you want to use it as the same array, simply pass it for

both.



Note: Queries, keys and values must either be all rank 3 or all rank 4 arrays. Mixing them doesn't work. The

output rank will depend on the input rank.

* **queries** - input 3D array "queries" of shape [batchSize, featureKeys, queryCount]
or 4D array of shape [batchSize, numHeads, featureKeys, queryCount] (NUMERIC type)
* **keys** - input 3D array "keys" of shape [batchSize, featureKeys, timesteps]
or 4D array of shape [batchSize, numHeads, featureKeys, timesteps] (NUMERIC type)
* **values** - input 3D array "values" of shape [batchSize, featureValues, timesteps]
or 4D array of shape [batchSize, numHeads, featureValues, timesteps] (NUMERIC type)
* **mask** - OPTIONAL; array that defines which values should be skipped of shape [batchSize, timesteps] (NUMERIC type)
* **scaled** - normalization, false -> do not apply normalization, true -> apply normalization

## <a name="dropout">dropout</a>
```JAVA
INDArray dropout(INDArray input, double inputRetainProbability)

SDVariable dropout(SDVariable input, double inputRetainProbability)
SDVariable dropout(String name, SDVariable input, double inputRetainProbability)
```
Dropout operation

* **input** - Input array (NUMERIC type)
* **inputRetainProbability** - Probability of retaining an input (set to 0 with probability 1-p)

## <a name="elu">elu</a>
```JAVA
INDArray elu(INDArray x)

SDVariable elu(SDVariable x)
SDVariable elu(String name, SDVariable x)
```
Element-wise exponential linear unit (ELU) function:

out = x if x > 0

out = a * (exp(x) - 1) if x <= 0

with constant a = 1.0

<p>

See: <a href="https://arxiv.org/abs/1511.07289">https://arxiv.org/abs/1511.07289</a>

* **x** - Input variable (NUMERIC type)

## <a name="gelu">gelu</a>
```JAVA
INDArray gelu(INDArray x)

SDVariable gelu(SDVariable x)
SDVariable gelu(String name, SDVariable x)
```
GELU activation function - Gaussian Error Linear Units

For more details, see <i>Gaussian Error Linear Units (GELUs)</i> - <a href="https://arxiv.org/abs/1606.08415">https://arxiv.org/abs/1606.08415</a>

This method uses the sigmoid approximation

* **x** - Input variable (NUMERIC type)

## <a name="hardSigmoid">hardSigmoid</a>
```JAVA
INDArray hardSigmoid(INDArray x)

SDVariable hardSigmoid(SDVariable x)
SDVariable hardSigmoid(String name, SDVariable x)
```
Element-wise hard sigmoid function:

out[i] = 0 if in[i] <= -2.5

out[1] = 0.2*in[i]+0.5 if -2.5 < in[i] < 2.5

out[i] = 1 if in[i] >= 2.5

* **x** - Input variable (NUMERIC type)

## <a name="hardTanh">hardTanh</a>
```JAVA
INDArray hardTanh(INDArray x)

SDVariable hardTanh(SDVariable x)
SDVariable hardTanh(String name, SDVariable x)
```
Element-wise hard tanh function:

out[i] = -1 if in[i] <= -1

out[1] = in[i] if -1 < in[i] < 1

out[i] = 1 if in[i] >= 1

* **x** - Input variable (NUMERIC type)

## <a name="hardTanhDerivative">hardTanhDerivative</a>
```JAVA
INDArray hardTanhDerivative(INDArray x)

SDVariable hardTanhDerivative(SDVariable x)
SDVariable hardTanhDerivative(String name, SDVariable x)
```
Derivative (dOut/dIn) of the element-wise hard Tanh function - hardTanh(INDArray)

* **x** - Input variable (NUMERIC type)

## <a name="layerNorm">layerNorm</a>
```JAVA
INDArray layerNorm(INDArray input, INDArray gain, INDArray bias, boolean channelsFirst, int[] dimensions)

SDVariable layerNorm(SDVariable input, SDVariable gain, SDVariable bias, boolean channelsFirst, int[] dimensions)
SDVariable layerNorm(String name, SDVariable input, SDVariable gain, SDVariable bias, boolean channelsFirst, int[] dimensions)
INDArray layerNorm(INDArray input, INDArray gain, boolean channelsFirst, int[] dimensions)

SDVariable layerNorm(SDVariable input, SDVariable gain, boolean channelsFirst, int[] dimensions)
SDVariable layerNorm(String name, SDVariable input, SDVariable gain, boolean channelsFirst, int[] dimensions)
```
Apply Layer Normalization



y = gain * standardize(x) + bias

* **input** - Input variable (NUMERIC type)
* **gain** - Gain (NUMERIC type)
* **bias** - Bias (NUMERIC type)
* **channelsFirst** - For 2D input - unused. True for NCHW (minibatch, channels, height, width), false for NHWC data
* **dimensions** - Dimensions to perform layer norm over - dimension=1 for 2d/MLP data, dimension=1,2,3 for CNNs (Size: AtLeast(min=1)
* **input** - Input variable (NUMERIC type)
* **gain** - Gain (NUMERIC type)
* **channelsFirst** - For 2D input - unused. True for NCHW (minibatch, channels, height, width), false for NHWC data
* **dimensions** - Dimensions to perform layer norm over - dimension=1 for 2d/MLP data, dimension=1,2,3 for CNNs (Size: AtLeast(min=1)

## <a name="leakyRelu">leakyRelu</a>
```JAVA
INDArray leakyRelu(INDArray x, double alpha)

SDVariable leakyRelu(SDVariable x, double alpha)
SDVariable leakyRelu(String name, SDVariable x, double alpha)
```
Element-wise leaky ReLU function:

out = x if x >= 0.0

out = alpha * x if x < cutoff

Alpha value is most commonly set to 0.01

* **x** - Input variable (NUMERIC type)
* **alpha** - Cutoff - commonly 0.01

## <a name="leakyReluDerivative">leakyReluDerivative</a>
```JAVA
INDArray leakyReluDerivative(INDArray x, double alpha)

SDVariable leakyReluDerivative(SDVariable x, double alpha)
SDVariable leakyReluDerivative(String name, SDVariable x, double alpha)
```
Leaky ReLU derivative: dOut/dIn given input.

* **x** - Input variable (NUMERIC type)
* **alpha** - Cutoff - commonly 0.01

## <a name="linear">linear</a>
```JAVA
INDArray linear(INDArray input, INDArray weights, INDArray bias)

SDVariable linear(SDVariable input, SDVariable weights, SDVariable bias)
SDVariable linear(String name, SDVariable input, SDVariable weights, SDVariable bias)
```
Linear layer operation: out = mmul(in,w) + bias

Note that bias array is optional

* **input** - Input data (NUMERIC type)
* **weights** - Weights variable, shape [nIn, nOut] (NUMERIC type)
* **bias** - Optional bias variable (may be null) (NUMERIC type)

## <a name="logSigmoid">logSigmoid</a>
```JAVA
INDArray logSigmoid(INDArray x)

SDVariable logSigmoid(SDVariable x)
SDVariable logSigmoid(String name, SDVariable x)
```
Element-wise sigmoid function: out[i] = log(sigmoid(in[i]))

* **x** - Input variable (NUMERIC type)

## <a name="logSoftmax">logSoftmax</a>
```JAVA
INDArray logSoftmax(INDArray x)

SDVariable logSoftmax(SDVariable x)
SDVariable logSoftmax(String name, SDVariable x)
```
Log softmax activation

* **x** -  (NUMERIC type)

## <a name="logSoftmax">logSoftmax</a>
```JAVA
INDArray logSoftmax(INDArray x, int dimension)

SDVariable logSoftmax(SDVariable x, int dimension)
SDVariable logSoftmax(String name, SDVariable x, int dimension)
```
Log softmax activation

* **x** - Input (NUMERIC type)
* **dimension** - Dimension along which to apply log softmax

## <a name="multiHeadDotProductAttention">multiHeadDotProductAttention</a>
```JAVA
INDArray multiHeadDotProductAttention(INDArray queries, INDArray keys, INDArray values, INDArray Wq, INDArray Wk, INDArray Wv, INDArray Wo, INDArray mask, boolean scaled)

SDVariable multiHeadDotProductAttention(SDVariable queries, SDVariable keys, SDVariable values, SDVariable Wq, SDVariable Wk, SDVariable Wv, SDVariable Wo, SDVariable mask, boolean scaled)
SDVariable multiHeadDotProductAttention(String name, SDVariable queries, SDVariable keys, SDVariable values, SDVariable Wq, SDVariable Wk, SDVariable Wv, SDVariable Wo, SDVariable mask, boolean scaled)
```
This performs multi-headed dot product attention on the given timeseries input

out = concat(head_1, head_2, ..., head_n) * Wo

head_i = dot_product_attention(Wq_i*q, Wk_i*k, Wv_i*v)



Optionally with normalization when calculating the attention for each head.



See also "Attention is all you need" (https://arxiv.org/abs/1706.03762, pp. 4,5, "3.2.2 Multi-Head Attention")



This makes use of dot_product_attention OP support for rank 4 inputs.

see dotProductAttention(INDArray, INDArray, INDArray, INDArray, boolean, boolean)

* **queries** - input 3D array "queries" of shape [batchSize, featureKeys, queryCount] (NUMERIC type)
* **keys** - input 3D array "keys" of shape [batchSize, featureKeys, timesteps] (NUMERIC type)
* **values** - input 3D array "values" of shape [batchSize, featureValues, timesteps] (NUMERIC type)
* **Wq** - input query projection weights of shape [numHeads, projectedKeys, featureKeys] (NUMERIC type)
* **Wk** - input key projection weights of shape [numHeads, projectedKeys, featureKeys] (NUMERIC type)
* **Wv** - input value projection weights of shape [numHeads, projectedValues, featureValues] (NUMERIC type)
* **Wo** - output projection weights of shape [numHeads * projectedValues, outSize] (NUMERIC type)
* **mask** - OPTIONAL; array that defines which values should be skipped of shape [batchSize, timesteps] (NUMERIC type)
* **scaled** - normalization, false -> do not apply normalization, true -> apply normalization

## <a name="pad">pad</a>
```JAVA
INDArray pad(INDArray input, INDArray padding, double constant)

SDVariable pad(SDVariable input, SDVariable padding, double constant)
SDVariable pad(String name, SDVariable input, SDVariable padding, double constant)
```
Padding operation 

* **input** - Input tensor (NUMERIC type)
* **padding** - Padding value (NUMERIC type)
* **constant** - Padding constant

## <a name="prelu">prelu</a>
```JAVA
INDArray prelu(INDArray input, INDArray alpha, int[] sharedAxes)

SDVariable prelu(SDVariable input, SDVariable alpha, int[] sharedAxes)
SDVariable prelu(String name, SDVariable input, SDVariable alpha, int[] sharedAxes)
```
PReLU (Parameterized Rectified Linear Unit) operation.  Like LeakyReLU with a learnable alpha:

out[i] = in[i] if in[i] >= 0

out[i] = in[i] * alpha[i] otherwise



sharedAxes allows you to share learnable parameters along axes.

For example, if the input has shape [batchSize, channels, height, width]

and you want each channel to have its own cutoff, use sharedAxes = [2, 3] and an

alpha with shape [channels].

* **input** - Input data (NUMERIC type)
* **alpha** - The cutoff variable.  Note that the batch dimension (the 0th, whether it is batch or not) should not be part of alpha. (NUMERIC type)
* **sharedAxes** - Which axes to share cutoff parameters along. (Size: AtLeast(min=1)

## <a name="relu">relu</a>
```JAVA
INDArray relu(INDArray x, double cutoff)

SDVariable relu(SDVariable x, double cutoff)
SDVariable relu(String name, SDVariable x, double cutoff)
```
Element-wise rectified linear function with specified cutoff:

out[i] = in[i] if in[i] >= cutoff

out[i] = 0 otherwise

* **x** - Input (NUMERIC type)
* **cutoff** - Cutoff value for ReLU operation - x > cutoff ? x : 0. Usually 0

## <a name="relu6">relu6</a>
```JAVA
INDArray relu6(INDArray x, double cutoff)

SDVariable relu6(SDVariable x, double cutoff)
SDVariable relu6(String name, SDVariable x, double cutoff)
```
Element-wise "rectified linear 6" function with specified cutoff:

out[i] = min(max(in, cutoff), 6)

* **x** - Input (NUMERIC type)
* **cutoff** - Cutoff value for ReLU operation. Usually 0

## <a name="reluLayer">reluLayer</a>
```JAVA
INDArray reluLayer(INDArray input, INDArray weights, INDArray bias)

SDVariable reluLayer(SDVariable input, SDVariable weights, SDVariable bias)
SDVariable reluLayer(String name, SDVariable input, SDVariable weights, SDVariable bias)
```
ReLU (Rectified Linear Unit) layer operation: out = relu(mmul(in,w) + bias)

Note that bias array is optional

* **input** - Input data (NUMERIC type)
* **weights** - Weights variable (NUMERIC type)
* **bias** - Optional bias variable (may be null) (NUMERIC type)

## <a name="selu">selu</a>
```JAVA
INDArray selu(INDArray x)

SDVariable selu(SDVariable x)
SDVariable selu(String name, SDVariable x)
```
Element-wise SeLU function - Scaled exponential Lineal Unit: see <a href="https://arxiv.org/abs/1706.02515">Self-Normalizing Neural Networks</a>



out[i] = scale * alpha * (exp(in[i])-1) if in[i]>0, or 0 if in[i] <= 0

Uses default scale and alpha values.

* **x** - Input variable (NUMERIC type)

## <a name="sigmoid">sigmoid</a>
```JAVA
INDArray sigmoid(INDArray x)

SDVariable sigmoid(SDVariable x)
SDVariable sigmoid(String name, SDVariable x)
```
Element-wise sigmoid function: out[i] = 1.0/(1+exp(-in[i]))

* **x** - Input variable (NUMERIC type)

## <a name="sigmoidDerivative">sigmoidDerivative</a>
```JAVA
INDArray sigmoidDerivative(INDArray x, INDArray wrt)

SDVariable sigmoidDerivative(SDVariable x, SDVariable wrt)
SDVariable sigmoidDerivative(String name, SDVariable x, SDVariable wrt)
```
Element-wise sigmoid function derivative: dL/dIn given input and dL/dOut

* **x** - Input Variable (NUMERIC type)
* **wrt** - Gradient at the output - dL/dOut. Must have same shape as the input (NUMERIC type)

## <a name="softmax">softmax</a>
```JAVA
INDArray softmax(INDArray x, int dimension)

SDVariable softmax(SDVariable x, int dimension)
SDVariable softmax(String name, SDVariable x, int dimension)
INDArray softmax(INDArray x)

SDVariable softmax(SDVariable x)
SDVariable softmax(String name, SDVariable x)
```
Softmax activation, along the specified dimension

* **x** - Input (NUMERIC type)
* **dimension** - Dimension along which to apply softmax - default = -1
* **x** - Input (NUMERIC type)

## <a name="softmaxDerivative">softmaxDerivative</a>
```JAVA
INDArray softmaxDerivative(INDArray x, INDArray wrt, int dimension)

SDVariable softmaxDerivative(SDVariable x, SDVariable wrt, int dimension)
SDVariable softmaxDerivative(String name, SDVariable x, SDVariable wrt, int dimension)
```
Softmax derivative function

* **x** - Softmax input (NUMERIC type)
* **wrt** - Gradient at output, dL/dx (NUMERIC type)
* **dimension** - Softmax dimension

## <a name="softplus">softplus</a>
```JAVA
INDArray softplus(INDArray x)

SDVariable softplus(SDVariable x)
SDVariable softplus(String name, SDVariable x)
```
Element-wise softplus function: out = log(exp(x) + 1)

* **x** - Input variable (NUMERIC type)

## <a name="softsign">softsign</a>
```JAVA
INDArray softsign(INDArray x)

SDVariable softsign(SDVariable x)
SDVariable softsign(String name, SDVariable x)
```
Element-wise softsign function: out = x / (abs(x) + 1)

* **x** - Input variable (NUMERIC type)

## <a name="softsignDerivative">softsignDerivative</a>
```JAVA
INDArray softsignDerivative(INDArray x)

SDVariable softsignDerivative(SDVariable x)
SDVariable softsignDerivative(String name, SDVariable x)
```
Element-wise derivative (dOut/dIn) of the softsign function softsign(INDArray)

* **x** - Input variable (NUMERIC type)

## <a name="swish">swish</a>
```JAVA
INDArray swish(INDArray x)

SDVariable swish(SDVariable x)
SDVariable swish(String name, SDVariable x)
```
Element-wise "swish" function: out = x * sigmoid(b*x) with b=1.0

See: <a href="https://arxiv.org/abs/1710.05941">https://arxiv.org/abs/1710.05941</a>

* **x** - Input variable (NUMERIC type)

## <a name="tanh">tanh</a>
```JAVA
INDArray tanh(INDArray x)

SDVariable tanh(SDVariable x)
SDVariable tanh(String name, SDVariable x)
```
Elementwise tanh (hyperbolic tangent) operation: out = tanh(x)

* **x** - Input variable (NUMERIC type)

