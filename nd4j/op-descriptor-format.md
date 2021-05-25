---
title: Overview
short_title: Overview
description: Model import framework overview and examples
category: Model import
weight: 1
---

An nd4j "op" is  an individual math operator (think add, subtract). This is best described in nd4j's [DynamicCustomOp](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ops/DynamicCustomOp.java). 
Given a set of input arguments as [ndarrays](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/linalg/api/ndarray/INDArray.java)

It then outputs ndarrays to be potentially passed to other ops for execution. Each op may have other input parameters such as booleans, doubles, floats, ints, and longs.

For performance reasons, output arrays may be passed in if the memory has already been pre allocated. Otherwise, each op can calculate the output shape of its outputs and dynamically generate the result ndarrays.

Every op has a calculateOutputShape implemented in c++ that's used to dynnamically create result arrays.
More information can be found in the [architecture decision record](https://github.com/eclipse/deeplearning4j/blob/7543fe9222dc8ecbf8d5a69097921cf5bb8e2f9d/ADRs/0003-Import_IR.md) for implementing this feature.


The op descriptor format is a serialized state of nd4j's operations. The current set of operations is saved in a protobuf format.
You can find the latest op set [here](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/samediff-import/samediff-import-api/src/main/resources/nd4j-op-def.pbtxt)

This op descriptor format is generated from a utility that parses the java and c++ code bases to automatically generate the set of op descriptors based on the code. It is possible for a user to run this utility to generate their own op descriptors as well.
If needed, the utility can be found [here](https://github.com/eclipse/deeplearning4j/tree/master/contrib/codegen-tools/libnd4j-gen)
It is a self contained maven project that you can import and run yourself. Note, that at this time we do not publish the tool on maven central. It is not really meant for end users, but please feel free to ask about it on our [forums](https://community.konduit.ai). The core protobuf op descriptor files can be found [here](https://github.com/eclipse/deeplearning4j/tree/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/protobuf/nd4j)

This is equivalent to an [onnx op](https://github.com/onnx/onnx/blob/master/docs/Operators.md) or a [tensroflow op](https://www.tensorflow.org/guide/create_op). "Ops" in deep learning frameworks are essentially math operations as simple as add or subtract or as complex as convolution that operate on input and output ndarrays. They also may contain numerical or string attributes as parameters for the execution of the operation.

Neural network execution in deep learning frameworks happens as a [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph). This allows a graph to be processed as a sequential list of operations.

For an end user, this means that every graph can be saved as a list of sequential steps for execution, eventually ending in 1 or more outputs.

The main concepts are fairly straightforward. A [Mapper](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/protobuf/nd4j/mapper.proto) maps input attributes and input/output ndarrays to their equivalents in nd4j.

Nd4j's graph format is a flatbuffers format. The method for export maybe found [here](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/autodiff/samediff/SameDiff.java#L4784)


