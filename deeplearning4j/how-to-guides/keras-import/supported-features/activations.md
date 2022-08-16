---
description: Supported Keras activations.
---

# Activations

We support all [Keras activation functions](https://keras.io/activations), namely:

* softmax
* elu
* selu
* softplus
* softsign
* relu
* tanh
* sigmoid
* hard\_sigmoid
* linear

The mapping of Keras to DL4J activation functions is defined in [KerasActivationUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasActivationUtils.java)
