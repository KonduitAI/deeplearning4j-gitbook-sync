---
title: RNN
short_title: RNN
description: 
category: Operations
weight: 50
---
# Operation classes
## gru
```JAVA
INDArray[] gru(INDArray x, INDArray hLast, GRUWeights gRUWeights)

SDVariable[] gru(SDVariable x, SDVariable hLast, GRUWeights gRUWeights)
SDVariable[] gru(String name, SDVariable x, SDVariable hLast, GRUWeights gRUWeights)
```
The GRU cell.  Does a single time step operation

* **x**  (NUMERIC type) - Input, with shape [batchSize, inSize]
* **hLast**  (NUMERIC type) - Output of the previous cell/time step, with shape [batchSize, numUnits]
* **GRUWeights** - see [GRUWeights](#GRUWeights)

## lstmCell
```JAVA
INDArray[] lstmCell(INDArray x, INDArray cLast, INDArray yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)

SDVariable[] lstmCell(SDVariable x, SDVariable cLast, SDVariable yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
SDVariable[] lstmCell(String name, SDVariable x, SDVariable cLast, SDVariable yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
```
The LSTM cell.  Does a single time step operation.

* **x**  (NUMERIC type) - Input, with shape [batchSize, inSize]
* **cLast**  (NUMERIC type) - Previous cell state, with shape [batchSize, numUnits]
* **yLast**  (NUMERIC type) - revious cell output, with shape [batchSize, numUnits]
* **LSTMWeights** - see [LSTMWeights](#LSTMWeights)
* **LSTMConfiguration** - see [LSTMConfiguration](#LSTMConfiguration)

## lstmLayer
```JAVA
INDArray[] lstmLayer(INDArray x, INDArray cLast, INDArray yLast, INDArray maxTSLength, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)

SDVariable[] lstmLayer(SDVariable x, SDVariable cLast, SDVariable yLast, SDVariable maxTSLength, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)
SDVariable[] lstmLayer(String name, SDVariable x, SDVariable cLast, SDVariable yLast, SDVariable maxTSLength, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)
INDArray[] lstmLayer(INDArray x, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)

SDVariable[] lstmLayer(SDVariable x, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)
SDVariable[] lstmLayer(String name, SDVariable x, LSTMLayerWeights lSTMLayerWeights, LSTMLayerConfig lSTMLayerConfig)
```
Long Short-Term Memory layer - Hochreiter 1997.

SUPPORTS following data formats:

for unidirectional:

TNS: shapes [timeLength, numExamples, inOutSize]

NST: shapes [numExamples, inOutSize, timeLength]

NTS: shapes [numExamples, timeLength, inOutSize]

for bidirectional:

T2NS: shapes [timeLength, 2, numExamples, inOutSize] (for ONNX)

SUPPORTS following direction modes:

FWD: forward

BWD: backward

BIDIR_SUM: bidirectional sum

BIDIR_CONCAT: bidirectional concat

BIDIR_EXTRA_DIM: bidirectional extra output dim (in conjunction with format dataFormat - T2NS)

You may use different gate configurations:

specify gate/cell/out aplha/beta and numbers of activations for gate/cell/out described in activations enum

("RELU","SIGMOID","AFFINE","LEAKY_RELU","THRESHHOLD_RELU","SCALED_TAHN","HARD_SIGMOID","ELU","SOFTSIGN","SOFTPLUS")

Also this layer supports MKLDNN (DNNL) and cuDNN acceleration

* **x**  (NUMERIC type) -  Input, with shape dependent on the data format (in config).
* **cLast**  (NUMERIC type) - Previous/initial cell state, with shape [batchSize, numUnits]
* **yLast**  (NUMERIC type) - Previous/initial cell output, with shape [batchSize, numUnits]
* **maxTSLength**  (NUMERIC type) - maxTSLength with shape [batchSize]
* **LSTMLayerWeights** - see [LSTMLayerWeights](#LSTMLayerWeights)
* **LSTMLayerConfig** - see [LSTMLayerConfig](#LSTMLayerConfig)

## lstmblock
```JAVA
INDArray lstmblock(INDArray maxTSLength, INDArray x, INDArray cLast, INDArray yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)

SDVariable lstmblock(SDVariable maxTSLength, SDVariable x, SDVariable cLast, SDVariable yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
SDVariable lstmblock(String name, SDVariable maxTSLength, SDVariable x, SDVariable cLast, SDVariable yLast, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
INDArray lstmblock(INDArray x, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)

SDVariable lstmblock(SDVariable x, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
SDVariable lstmblock(String name, SDVariable x, LSTMWeights lSTMWeights, LSTMConfiguration lSTMConfiguration)
```
The LSTM block

* **maxTSLength**  (NUMERIC type) - 
* **x**  (NUMERIC type) -  Input, with shape dependent on the data format (in config).
* **cLast**  (NUMERIC type) - Previous/initial cell state, with shape [batchSize, numUnits]
* **yLast**  (NUMERIC type) - Previous/initial cell output, with shape [batchSize, numUnits]
* **LSTMWeights** - see [LSTMWeights](#LSTMWeights)
* **LSTMConfiguration** - see [LSTMConfiguration](#LSTMConfiguration)

## sru
```JAVA
INDArray sru(INDArray x, INDArray initialC, INDArray mask, SRUWeights sRUWeights)

SDVariable sru(SDVariable x, SDVariable initialC, SDVariable mask, SRUWeights sRUWeights)
SDVariable sru(String name, SDVariable x, SDVariable initialC, SDVariable mask, SRUWeights sRUWeights)
INDArray sru(INDArray x, INDArray initialC, SRUWeights sRUWeights)

SDVariable sru(SDVariable x, SDVariable initialC, SRUWeights sRUWeights)
SDVariable sru(String name, SDVariable x, SDVariable initialC, SRUWeights sRUWeights)
```
The SRU layer.  Does a single time step operation.

* **x**  (NUMERIC type) - Input, with shape [batchSize, inSize]
* **initialC**  (NUMERIC type) - Initial cell state, with shape [batchSize, inSize]
* **mask**  (NUMERIC type) - An optional dropout mask, with shape [batchSize, inSize]
* **SRUWeights** - see [SRUWeights](#SRUWeights)

## sruCell
```JAVA
INDArray sruCell(INDArray x, INDArray cLast, SRUWeights sRUWeights)

SDVariable sruCell(SDVariable x, SDVariable cLast, SRUWeights sRUWeights)
SDVariable sruCell(String name, SDVariable x, SDVariable cLast, SRUWeights sRUWeights)
```
The SRU layer.  Does a single time step operation.

* **x**  (NUMERIC type) - Input, with shape [batchSize, inSize]
* **cLast**  (NUMERIC type) - Previous cell state, with shape [batchSize, inSize]
* **SRUWeights** - see [SRUWeights](#SRUWeights)

# Configuration Classes
## LSTMConfiguration
* **RnnDataFormat** -  The data format of the input. Input shape depends on data format (in config):<br>
 TNS -> [timeSteps, batchSize, inSize]<br>
 NST -> [batchSize, inSize, timeSteps]<br>
 NTS -> [batchSize, timeSteps, inSize]<br> (ENUM type)
* **peepHole** - Whether to provide peephole connections (BOOL type)
* **forgetBias** - The bias added to forget gates in order to reduce the scale of forgetting in the beginning of the training. (NUMERIC type)
* **clippingCellValue** - The bias added to forget gates in order to reduce the scale of forgetting in the beginning of the training. (NUMERIC type)

Used in these ops: 
[lstmCell](#lstmCell)
[lstmblock](#lstmblock)
## LSTMLayerConfig
* **LSTMDataFormat** - for unidirectional:  TNS: shape [timeLength, numExamples, inOutSize] - sometimes referred to as "time major"<br>
  NST: shape [numExamples, inOutSize, timeLength]<br>
  NTS: shape [numExamples, timeLength, inOutSize] - TF "time_major=false" layout<br> for bidirectional:
   T2NS: 3 = [timeLength, 2, numExamples, inOutSize] (for ONNX) (ENUM type)
* **LSTMDirectionMode** - direction <br>
 FWD: 0 = fwd
 BWD: 1 = bwd
 BIDIR_SUM: 2 = bidirectional sum
 BIDIR_CONCAT: 3 = bidirectional concat
 BIDIR_EXTRA_DIM: 4 = bidirectional extra output dim (in conjunction with format dataFormat = 3) (ENUM type)
* **gateAct** - Activations (ENUM type)
* **cellAct** - Activations (ENUM type)
* **outAct** - Activations (ENUM type)
* **retFullSequence** - indicates whether to return whole time sequence h {h_0, h_1, ... , h_sL-1} (BOOL type) - default = true
* **retLastH** - indicates whether to return output at last time step only,
 in this case shape would be [bS, nOut] (exact shape depends on dataFormat argument) (BOOL type) - default = false
* **retLastC** - indicates whether to return cells state at last time step only,
 in this case shape would be [bS, nOut] (exact shape depends on dataFormat argument) (BOOL type) - default = false
* **cellClip** - Cell clipping value, if it = 0 then do not apply clipping (NUMERIC type) - default = 0.0
* **gateAlpha** - null (NUMERIC type) - default = 0.0
* **gateBeta** - null (NUMERIC type) - default = 0.0
* **cellAlpha** - null (NUMERIC type) - default = 0.0
* **cellBeta** - null (NUMERIC type) - default = 0.0
* **outAlpha** - null (NUMERIC type) - default = 0.0
* **outBeta** - null (NUMERIC type) - default = 0.0

Used in these ops: 
[lstmLayer](#lstmLayer)
## GRUWeights
* **ruWeight**- null (NUMERIC type)
* **cWeight**- null (NUMERIC type)
* **ruBias**- null (NUMERIC type)
* **cBias**- null (NUMERIC type)

Used in these ops: 
[gru](#gru)
## SRUWeights
* **weights**- null (NUMERIC type)
* **bias**- null (NUMERIC type)

Used in these ops: 
[sru](#sru)
[sruCell](#sruCell)
## LSTMWeights
* **ruWeight**- null (NUMERIC type)
* **inputPeepholeWeights**- null (NUMERIC type)
* **forgetPeepholeWeights**- null (NUMERIC type)
* **outputPeepholeWeights**- null (NUMERIC type)
* **bias**- null (NUMERIC type)

Used in these ops: 
[lstmCell](#lstmCell)
[lstmblock](#lstmblock)
## LSTMLayerWeights
* **inputWeights**- input weights Wx:
 1) shapes [nIn, 4*nOut] for FWD,BWD  2) shapes [2, nIn, 4*nOut] BIDIR_SUM, BIDIR_CONCAT and BIDIR_EXTRA_DIM (NUMERIC type)
* **recurrentWeights**- // recurrent weights Wr:
 1) shapes[nIn, 4*nOut] for FWD, BWD  2) shapes [2, nIn, 4*nOut] BIDIR_SUM, BIDIR_CONCAT and BIDIR_EXTRA_DIM (NUMERIC type)
* **biases**- biases
 1) shapes [4*nOut] for FWD, BWD  2) shapes [2, 4*nOut] for BIDIR_SUM, BIDIR_CONCAT and BIDIR_EXTRA_DIM (NUMERIC type)
* **peepholeWeights**- peephole weights Wp:
  1) [3*nOut]    when directionMode <  2
  2) [2, 3*nOut] when directionMode >= 2 (NUMERIC type)

Used in these ops: 
[lstmLayer](#lstmLayer)
