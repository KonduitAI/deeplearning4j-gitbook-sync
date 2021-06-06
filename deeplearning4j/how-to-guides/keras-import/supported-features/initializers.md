---
title: Keras Initializers
short_title: Initializers
description: Supported Keras weight initializers.
category: Keras Import
weight: 4
---

# Initializers

DL4J supports all available [Keras initializers](https://keras.io/initializers), namely:

*  Zeros
*  Ones
*  Constant
*  RandomNormal
*  RandomUniform
*  TruncatedNormal
*  VarianceScaling
*  Orthogonal
*  Identity
*  lecun\_uniform
*  lecun\_normal
*  glorot\_normal
*  glorot\_uniform
*  he\_normal
*  he\_uniform

The mapping of Keras to DL4J initializers can be found in [KerasInitilizationUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasInitilizationUtils.java).

