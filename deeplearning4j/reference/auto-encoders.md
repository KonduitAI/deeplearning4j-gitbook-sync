# Auto Encoders

## What are autoencoders?

Autoencoders are neural networks for unsupervised learning. Eclipse Deeplearning4j supports certain autoencoder layers such as variational autoencoders.

## Where’s Restricted Boltzmann Machine?

RBMs are no longer supported as of version 0.9.x. They are no longer best-in-class for most machine learning problems.

## Supported layers

### AutoEncoder

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/AutoEncoder.java)

Autoencoder layer. Adds noise to input and learn a reconstruction function.

**corruptionLevel**

```text
public Builder corruptionLevel(double corruptionLevel)
```

Level of corruption - 0.0 \(none\) to 1.0 \(all values corrupted\)

**sparsity**

```text
public Builder sparsity(double sparsity)
```

Autoencoder sparity parameter

* param sparsity Sparsity

### VariationalAutoencoder

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/variational/VariationalAutoencoder.java)

Variational Autoencoder layer

See: Kingma & Welling, 2013: Auto-Encoding Variational Bayes - [https://arxiv.org/abs/1312.6114](https://arxiv.org/abs/1312.6114)

This implementation allows multiple encoder and decoder layers, the number and sizes of which can be set independently.

A note on scores during pretraining: This implementation minimizes the negative of the variational lower bound objective as described in Kingma & Welling; the mathematics in that paper is based on maximization of the variational lower bound instead. Thus, scores reported during pretraining in DL4J are the negative of the variational lower bound equation in the paper. The backpropagation and learning procedure is otherwise as described there.

**encoderLayerSizes**

```text
public Builder encoderLayerSizes(int... encoderLayerSizes)
```

Size of the encoder layers, in units. Each encoder layer is functionally equivalent to a {- link org.deeplearning4j.nn.conf.layers.DenseLayer}. Typically the number and size of the decoder layers \(set via {- link \#decoderLayerSizes\(int…\)} is similar to the encoder layers.

**setEncoderLayerSizes**

```text
public void setEncoderLayerSizes(int... encoderLayerSizes)
```

Size of the encoder layers, in units. Each encoder layer is functionally equivalent to a {- link org.deeplearning4j.nn.conf.layers.DenseLayer}. Typically the number and size of the decoder layers \(set via {- link \#decoderLayerSizes\(int…\)} is similar to the encoder layers.

* param encoderLayerSizes Size of each encoder layer in the variational autoencoder

**decoderLayerSizes**

```text
public Builder decoderLayerSizes(int... decoderLayerSizes)
```

Size of the decoder layers, in units. Each decoder layer is functionally equivalent to a {- link org.deeplearning4j.nn.conf.layers.DenseLayer}. Typically the number and size of the decoder layers is similar to the encoder layers \(set via {- link \#encoderLayerSizes\(int…\)}.

* param decoderLayerSizes Size of each deccoder layer in the variational autoencoder

**setDecoderLayerSizes**

```text
public void setDecoderLayerSizes(int... decoderLayerSizes)
```

Size of the decoder layers, in units. Each decoder layer is functionally equivalent to a {- link org.deeplearning4j.nn.conf.layers.DenseLayer}. Typically the number and size of the decoder layers is similar to the encoder layers \(set via {- link \#encoderLayerSizes\(int…\)}.

* param decoderLayerSizes Size of each deccoder layer in the variational autoencoder

**reconstructionDistribution**

```text
public Builder reconstructionDistribution(ReconstructionDistribution distribution)
```

The reconstruction distribution for the data given the hidden state - i.e., P\(data\|Z\).  
This should be selected carefully based on the type of data being modelled. For example:

* {- link GaussianReconstructionDistribution} + {identity or tanh} for real-valued \(Gaussian\) data  
* {- link BernoulliReconstructionDistribution} + sigmoid for binary-valued \(0 or 1\) data  
* param distribution Reconstruction distribution

**lossFunction**

```text
public Builder lossFunction(IActivation outputActivationFn, LossFunctions.LossFunction lossFunction)
```

Configure the VAE to use the specified loss function for the reconstruction, instead of a ReconstructionDistribution. Note that this is NOT following the standard VAE design \(as per Kingma & Welling\), which assumes a probabilistic output - i.e., some p\(x\|z\). It is however a valid network configuration, allowing for optimization of more traditional objectives such as mean squared error.  
Note: clearly, setting the loss function here will override any previously set recontruction distribution

* param outputActivationFn Activation function for the output/reconstruction
* param lossFunction Loss function to use

**lossFunction**

```text
public Builder lossFunction(Activation outputActivationFn, LossFunctions.LossFunction lossFunction)
```

Configure the VAE to use the specified loss function for the reconstruction, instead of a ReconstructionDistribution. Note that this is NOT following the standard VAE design \(as per Kingma & Welling\), which assumes a probabilistic output - i.e., some p\(x\|z\). It is however a valid network configuration, allowing for optimization of more traditional objectives such as mean squared error.  
Note: clearly, setting the loss function here will override any previously set recontruction distribution

* param outputActivationFn Activation function for the output/reconstruction
* param lossFunction Loss function to use

**lossFunction**

```text
public Builder lossFunction(IActivation outputActivationFn, ILossFunction lossFunction)
```

Configure the VAE to use the specified loss function for the reconstruction, instead of a ReconstructionDistribution. Note that this is NOT following the standard VAE design \(as per Kingma & Welling\), which assumes a probabilistic output - i.e., some p\(x\|z\). It is however a valid network configuration, allowing for optimization of more traditional objectives such as mean squared error.  
Note: clearly, setting the loss function here will override any previously set recontruction distribution

* param outputActivationFn Activation function for the output/reconstruction
* param lossFunction Loss function to use

**pzxActivationFn**

```text
public Builder pzxActivationFn(IActivation activationFunction)
```

Activation function for the input to P\(z\|data\).  
Care should be taken with this, as some activation functions \(relu, etc\) are not suitable due to being bounded in range \[0,infinity\).

* param activationFunction Activation function for p\(z\| x\)

**pzxActivationFunction**

```text
public Builder pzxActivationFunction(Activation activation)
```

Activation function for the input to P\(z\|data\).  
Care should be taken with this, as some activation functions \(relu, etc\) are not suitable due to being bounded in range \[0,infinity\).

* param activation Activation function for p\(z \| x\)

**nOut**

```text
public Builder nOut(int nOut)
```

Set the size of the VAE state Z. This is the output size during standard forward pass, and the size of the distribution P\(Z\|data\) during pretraining.

* param nOut Size of P\(Z \| data\) and output size

**numSamples**

```text
public Builder numSamples(int numSamples)
```

Set the number of samples per data point \(from VAE state Z\) used when doing pretraining. Default value: 1.

This is parameter L from Kingma and Welling: “In our experiments we found that the number of samples L per datapoint can be set to 1 as long as the minibatch size M was large enough, e.g. M = 100.”

* param numSamples Number of samples per data point for pretraining

