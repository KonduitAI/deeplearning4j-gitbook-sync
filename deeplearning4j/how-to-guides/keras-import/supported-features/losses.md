---
description: Supported Keras loss functions.
---

# Losses

DL4J supports all available [Keras losses](https://keras.io/losses) (except for `logcosh`), namely:

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

The mapping of Keras loss functions can be found in [KerasLossUtils](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-modelimport/src/main/java/org/deeplearning4j/nn/modelimport/keras/utils/KerasLossUtils.java).
