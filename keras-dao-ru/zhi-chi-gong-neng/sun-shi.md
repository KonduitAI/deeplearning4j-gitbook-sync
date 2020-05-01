---
description: 支持的损失函数
---

# 损失

### 支持的损失函数

DL4J支持所有可用的[Keras损失函数](https://keras.io/losses)（除了logcosh），即：

* mean\_squared\_error
* mean\_absolute\_error
* mean\_absolute\_percentage\_error
* mean\_squared\_logarithmic\_error
* squared\_hinge
* hinge
* categorical\_hinge
* logcosh
* categorical\_crossentropy
* sparse\_categorical\_crossentropy
* binary\_crossentropy
* kullback\_leibler\_divergence
* poisson
* cosine\_proximity

Keras的损失函数映射可在[KerasLossUtils](https://github.com/deeplearning4j/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasLossUtils.java)中找到。

