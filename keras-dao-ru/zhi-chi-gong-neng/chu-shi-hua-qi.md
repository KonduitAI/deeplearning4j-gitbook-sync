---
description: 支持的Keras权重初始化器
---

# 初始化器

### 支持的初始化器

DL4J 支持所有可用的 [Keras 初始化器](https://keras.io/initializers), 名称为:

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

从Keras 到 DL4J的初始化器映射可以从 [KerasInitilizationUtils](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasInitilizationUtils.java)中找到。

