---
description: 已支持的Keras约束。
---

# 约束

所有的 [Keras 约束](https://keras.io/constraints) 都被支持:

* max\_norm
* non\_neg
* unit\_norm
* min\_max\_norm

在 [KerasConstraintUtils](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasConstraintUtils.java)中实现Keras 到 DL4J 约束映射。

