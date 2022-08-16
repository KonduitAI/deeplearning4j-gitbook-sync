# RNN

## Operation classes

### gru

```java
INDArray gru(INDArray x, INDArray hLast, INDArray Wx, INDArray Wh, INDArray biases)

SDVariable gru(SDVariable x, SDVariable hLast, SDVariable Wx, SDVariable Wh, SDVariable biases)
SDVariable gru(String name, SDVariable x, SDVariable hLast, SDVariable Wx, SDVariable Wh, SDVariable biases)
```

The GRU operation. Gated Recurrent Unit - Cho et al. 2014.

* **x**  (NUMERIC) - input \[time, bS, nIn]
* **hLast**  (NUMERIC) - initial cell output (at time step = 0) \[bS, nOut]
* **Wx**  (NUMERIC) - input-to-hidden  weights, \[nIn, 3\*nOut]
* **Wh**  (NUMERIC) - hidden-to-hidden weights, \[nOut, 3\*nOut]
* **biases**  (NUMERIC) - biases, \[3\*nOut]

### gruCell

```java
INDArray[] gruCell(INDArray x, INDArray hLast, GRUWeights gRUWeights)

SDVariable[] gruCell(SDVariable x, SDVariable hLast, GRUWeights gRUWeights)
SDVariable[] gruCell(String name, SDVariable x, SDVariable hLast, GRUWeights gRUWeights)
```

The GRU cell. Does a single time step operation

* **x**  (NUMERIC) - Input, with shape \[batchSize, inSize]
* **hLast**  (NUMERIC) - Output of the previous cell/time step, with shape \[batchSize, numUnits]
* **GRUWeights** - see [GRUWeights](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#gruweights)

### lstmCell

```java
INDArray[] lstmCell(INDArray x, INDArray cLast, INDArray yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)

SDVariable[] lstmCell(SDVariable x, SDVariable cLast, SDVariable yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
SDVariable[] lstmCell(String name, SDVariable x, SDVariable cLast, SDVariable yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
```

The LSTM cell. Does a single time step operation.

* **x**  (NUMERIC) - Input, with shape \[batchSize, inSize]
* **cLast**  (NUMERIC) - Previous cell state, with shape \[batchSize, numUnits]
* **yLast**  (NUMERIC) - revious cell output, with shape \[batchSize, numUnits]
* **LSTMWeights** - see [LSTMWeights](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmweights)
* **LSTMConfiguration** - see [LSTMConfiguration](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmconfiguration)

### lstmLayer

```java
INDArray[] lstmLayer(INDArray x, INDArray cLast, INDArray yLast, INDArray maxTSLength, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)
INDArray[] lstmLayer(INDArray x, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)

SDVariable[] lstmLayer(SDVariable x, SDVariable cLast, SDVariable yLast, SDVariable maxTSLength, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)
SDVariable[] lstmLayer(SDVariable x, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)
SDVariable[] lstmLayer(String name, SDVariable x, SDVariable cLast, SDVariable yLast, SDVariable maxTSLength, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)
SDVariable[] lstmLayer(String name, SDVariable x, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)
```

Long Short-Term Memory layer - Hochreiter 1997.

SUPPORTS following data formats:

for unidirectional:

TNS: shapes \[timeLength, numExamples, inOutSize]

NST: shapes \[numExamples, inOutSize, timeLength]

NTS: shapes \[numExamples, timeLength, inOutSize]

for bidirectional:

T2NS: shapes \[timeLength, 2, numExamples, inOutSize] (for ONNX)

SUPPORTS following direction modes:

FWD: forward

BWD: backward

BIDIR\_SUM: bidirectional sum

BIDIR\_CONCAT: bidirectional concat

BIDIR\_EXTRA\_DIM: bidirectional extra output dim (in conjunction with format dataFormat - T2NS)

You may use different gate configurations:

specify gate/cell/out aplha/beta and numbers of activations for gate/cell/out described in activations enum

("RELU","SIGMOID","AFFINE","LEAKY\_RELU","THRESHHOLD\_RELU","SCALED\_TAHN","HARD\_SIGMOID","ELU","SOFTSIGN","SOFTPLUS")

Also this layer supports MKLDNN (DNNL) and cuDNN acceleration

* **x**  (NUMERIC) -  Input, with shape dependent on the data format (in config).
* **cLast**  (NUMERIC) - Previous/initial cell state, with shape \[batchSize, numUnits]
* **yLast**  (NUMERIC) - Previous/initial cell output, with shape \[batchSize, numUnits]
* **maxTSLength**  (NUMERIC) - maxTSLength with shape \[batchSize]
* **LSTMLayerWeights** - see [LSTMLayerWeights](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmlayerweights)
* **LSTMLayerConfig** - see [LSTMLayerConfig](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmlayerconfig)

### lstmblock

```java
INDArray lstmblock(INDArray maxTSLength, INDArray x, INDArray cLast, INDArray yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
INDArray lstmblock(INDArray x, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)

SDVariable lstmblock(SDVariable maxTSLength, SDVariable x, SDVariable cLast, SDVariable yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
SDVariable lstmblock(SDVariable x, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
SDVariable lstmblock(String name, SDVariable maxTSLength, SDVariable x, SDVariable cLast, SDVariable yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
SDVariable lstmblock(String name, SDVariable x, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
```

The LSTM block

* **maxTSLength**  (NUMERIC) -&#x20;
* **x**  (NUMERIC) -  Input, with shape dependent on the data format (in config).
* **cLast**  (NUMERIC) - Previous/initial cell state, with shape \[batchSize, numUnits]
* **yLast**  (NUMERIC) - Previous/initial cell output, with shape \[batchSize, numUnits]
* **LSTMWeights** - see [LSTMWeights](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmweights)
* **LSTMConfiguration** - see [LSTMConfiguration](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmconfiguration)

### sru

```java
INDArray sru(INDArray x, INDArray initialC, INDArray mask, SRUWeights sRUWeights)
INDArray sru(INDArray x, INDArray initialC, SRUWeights sRUWeights)

SDVariable sru(SDVariable x, SDVariable initialC, SDVariable mask, SRUWeights sRUWeights)
SDVariable sru(SDVariable x, SDVariable initialC, SRUWeights sRUWeights)
SDVariable sru(String name, SDVariable x, SDVariable initialC, SDVariable mask, SRUWeights sRUWeights)
SDVariable sru(String name, SDVariable x, SDVariable initialC, SRUWeights sRUWeights)
```

The SRU layer. Does a single time step operation.

* **x**  (NUMERIC) - Input, with shape \[batchSize, inSize]
* **initialC**  (NUMERIC) - Initial cell state, with shape \[batchSize, inSize]
* **mask**  (NUMERIC) - An optional dropout mask, with shape \[batchSize, inSize]
* **SRUWeights** - see [SRUWeights](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#sruweights)

### sruCell

```java
INDArray sruCell(INDArray x, INDArray cLast, SRUWeights sRUWeights)

SDVariable sruCell(SDVariable x, SDVariable cLast, SRUWeights sRUWeights)
SDVariable sruCell(String name, SDVariable x, SDVariable cLast, SRUWeights sRUWeights)
```

The SRU layer. Does a single time step operation.

* **x**  (NUMERIC) - Input, with shape \[batchSize, inSize]
* **cLast**  (NUMERIC) - Previous cell state, with shape \[batchSize, inSize]
* **SRUWeights** - see [SRUWeights](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#sruweights)

## Configuration Classes

### LSTMConfiguration

* **RnnDataFormat** (ENUM) -  The data format of the input. Input shape depends on data format (in config): &#x20;

TNS -> \[timeSteps, batchSize, inSize]

NST -> \[batchSize, inSize, timeSteps]

NTS -> \[batchSize, timeSteps, inSize]

* **peepHole** (BOOL) - Whether to provide peephole connections
* **forgetBias** (NUMERIC) - The bias added to forget gates in order to reduce the scale of forgetting in the beginning of the training.
* **clippingCellValue** (NUMERIC) - The bias added to forget gates in order to reduce the scale of forgetting in the beginning of the training.

Used in these ops: [lstmCell](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmcell) [lstmblock](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmblock)

### LSTMLayerConfig

* **LSTMDataFormat** (ENUM) - for unidirectional:  TNS: shape \[timeLength, numExamples, inOutSize] - sometimes referred to as "time major" &#x20;

NST: shape \[numExamples, inOutSize, timeLength]

NTS: shape \[numExamples, timeLength, inOutSize] - TF "time\_major=false" layout\
for bidirectional:

T2NS: 3 = \[timeLength, 2, numExamples, inOutSize] (for ONNX)

* **LSTMDirectionMode** (ENUM) - direction  &#x20;

FWD: 0 = fwd

BWD: 1 = bwd

BIDIR\_SUM: 2 = bidirectional sum

BIDIR\_CONCAT: 3 = bidirectional concat

BIDIR\_EXTRA\_DIM: 4 = bidirectional extra output dim (in conjunction with format dataFormat = 3)

* **gateAct** (ENUM) - Activations
* **cellAct** (ENUM) - Activations
* **outAct** (ENUM) - Activations
* **retFullSequence** (BOOL) - indicates whether to return whole time sequence h {h\_0, h\_1, ... , h\_sL-1} - default = true
*   **retLastH** (BOOL) - indicates whether to return output at last time step only,

    in this case shape would be \[bS, nOut] (exact shape depends on dataFormat argument) - default = false
*   **retLastC** (BOOL) - indicates whether to return cells state at last time step only,

    in this case shape would be \[bS, nOut] (exact shape depends on dataFormat argument) - default = false
* **cellClip** (NUMERIC) - Cell clipping value, if it = 0 then do not apply clipping - default = 0.0
* **gateAlpha** (NUMERIC) - null - default = 0.0
* **gateBeta** (NUMERIC) - null - default = 0.0
* **cellAlpha** (NUMERIC) - null - default = 0.0
* **cellBeta** (NUMERIC) - null - default = 0.0
* **outAlpha** (NUMERIC) - null - default = 0.0
* **outBeta** (NUMERIC) - null - default = 0.0

Used in these ops: [lstmLayer](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmlayer)

### GRUWeights

* **ruWeight**- null (NUMERIC type)
* **cWeight**- null (NUMERIC type)
* **ruBias**- null (NUMERIC type)
* **cBias**- null (NUMERIC type)

Used in these ops: [gruCell](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#grucell)

### SRUWeights

* **weights**- null (NUMERIC type)
* **bias**- null (NUMERIC type)

Used in these ops: [sru](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#sru) [sruCell](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#srucell)

### LSTMWeights

* **ruWeight**- null (NUMERIC type)
* **inputPeepholeWeights**- null (NUMERIC type)
* **forgetPeepholeWeights**- null (NUMERIC type)
* **outputPeepholeWeights**- null (NUMERIC type)
* **bias**- null (NUMERIC type)

Used in these ops: [lstmCell](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmcell) [lstmblock](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmblock)

### LSTMLayerWeights

*   **inputWeights**- input weights Wx:

    1\) shapes `[nIn, 4*nOut]` for FWD,BWD 2) shapes `[2, nIn, 4*nOut]` BIDIR\_SUM, BIDIR\_CONCAT and BIDIR\_EXTRA\_DIM (NUMERIC type)
*   **recurrentWeights**- recurrent weights Wr:

    1\) shapes `[nIn, 4*nOut]` for FWD, BWD 2) shapes `[2, nIn, 4*nOut]` BIDIR\_SUM, BIDIR\_CONCAT and BIDIR\_EXTRA\_DIM (NUMERIC type)
*   **biases**- biases

    1\) shapes `[4*nOut]` for FWD, BWD 2) shapes `[2, 4*nOut]` for BIDIR\_SUM, BIDIR\_CONCAT and BIDIR\_EXTRA\_DIM (NUMERIC type)
*   **peepholeWeights**- peephole weights Wp:

    1\) `[3*nOut]` when directionMode < 2

    2\) `[2, 3*nOut]` when directionMode >= 2 (NUMERIC type)

Used in these ops: [lstmLayer](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/samediff/reference/operation-namespaces/rnn.md#lstmlayer)
