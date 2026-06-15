---
title: "Autoencoders"
description: "Autoencoder and Variational Autoencoder layers in Deeplearning4j — architecture, configuration, and training"
---

# Autoencoders

Autoencoders are neural networks trained to reconstruct their inputs through a compressed latent representation. Eclipse Deeplearning4j supports a denoising AutoEncoder layer and a full VariationalAutoencoder (VAE) layer with configurable reconstruction distributions.

> **Note:** Restricted Boltzmann Machines (RBMs) were removed in version 0.9.x and are no longer supported.

## AutoEncoder Layer

The `AutoEncoder` layer (`org.deeplearning4j.nn.conf.layers.AutoEncoder`) is a denoising autoencoder. It adds random noise (corruption) to the input during training, then learns to reconstruct the clean original. This forces the network to learn a robust representation.

### Key Parameters

| Parameter | Method | Description |
|---|---|---|
| Corruption level | `corruptionLevel(double)` | Fraction of input values zeroed out during training. Range 0.0 (none) to 1.0 (all). Typical: 0.3 |
| Sparsity | `sparsity(double)` | Sparsity regularization penalty. Encourages few active hidden units. |

Standard layer parameters (`nIn`, `nOut`, `activation`, `weightInit`, `updater`, etc.) all apply.

### Configuration Example

```java
import org.deeplearning4j.nn.conf.MultiLayerConfiguration;
import org.deeplearning4j.nn.conf.NeuralNetConfiguration;
import org.deeplearning4j.nn.conf.layers.AutoEncoder;
import org.deeplearning4j.nn.conf.layers.OutputLayer;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.lossfunctions.LossFunctions;

int inputSize  = 784;  // e.g. MNIST 28x28
int hiddenSize = 256;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .updater(new org.nd4j.linalg.learning.config.Adam(1e-3))
    .list()
    .layer(new AutoEncoder.Builder()
        .nIn(inputSize).nOut(hiddenSize)
        .activation(Activation.RELU)
        .corruptionLevel(0.3)
        .sparsity(0.0)
        .build())
    // Tie weights back to reconstruction by adding a second AutoEncoder layer reversed,
    // or simply use a DenseLayer + OutputLayer for the decoder portion:
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.MSE)
        .nIn(hiddenSize).nOut(inputSize)
        .activation(Activation.SIGMOID)
        .build())
    .build();

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
```

During training the model minimises reconstruction loss on the corrupted-then-decoded output.

---

## VariationalAutoencoder Layer

The `VariationalAutoencoder` layer (`org.deeplearning4j.nn.conf.layers.variational.VariationalAutoencoder`) implements the VAE described in Kingma & Welling (2013), "Auto-Encoding Variational Bayes". It supports multiple encoder and decoder hidden layers, a configurable latent space size, and several reconstruction distributions.

Key ideas:

- The encoder maps input x to a distribution q(z|x) over latent variable z.
- A latent code z is sampled from q(z|x).
- The decoder maps z back to a reconstruction p(x|z).
- The training objective maximises the variational lower bound (ELBO).

> **Score sign convention:** DL4J minimises the *negative* of the variational lower bound, so reported scores during pretraining are negative values of the ELBO described in the paper.

### Builder Parameters

| Method | Description |
|---|---|
| `encoderLayerSizes(int...)` | Sizes of hidden layers in the encoder. Each acts like a `DenseLayer`. |
| `decoderLayerSizes(int...)` | Sizes of hidden layers in the decoder. Typically mirrors the encoder. |
| `nOut(int)` | Size of the latent space Z. |
| `reconstructionDistribution(ReconstructionDistribution)` | Distribution used to model p(x\|z). See distributions below. |
| `pzxActivationFunction(Activation)` | Activation for the mean/log-variance output feeding into p(z\|x). Avoid bounded activations like `RELU`. Use `TANH` or `IDENTITY`. |
| `numSamples(int)` | Number of latent samples per data point during pretraining (default 1). |
| `lossFunction(IActivation, ILossFunction)` | Alternative: use a deterministic loss function instead of a reconstruction distribution. |

### Configuration Example

```java
import org.deeplearning4j.nn.conf.layers.variational.VariationalAutoencoder;
import org.deeplearning4j.nn.conf.layers.variational.GaussianReconstructionDistribution;
import org.nd4j.linalg.activations.Activation;

int inputDim  = 784;
int latentDim = 32;

MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .updater(new org.nd4j.linalg.learning.config.Adam(1e-3))
    .list()
    .layer(new VariationalAutoencoder.Builder()
        .nIn(inputDim)
        .nOut(latentDim)                      // latent space size
        .encoderLayerSizes(512, 256)           // two encoder hidden layers
        .decoderLayerSizes(256, 512)           // two decoder hidden layers (mirrored)
        .pzxActivationFunction(Activation.IDENTITY)
        .reconstructionDistribution(
            new GaussianReconstructionDistribution(Activation.IDENTITY))
        .activation(Activation.LEAKYRELU)
        .build())
    .build();

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();
```

---

## Reconstruction Distributions

The reconstruction distribution defines how the decoder output is interpreted when computing the reconstruction loss. Choose based on the nature of your data.

### GaussianReconstructionDistribution

Models each output dimension as an independent Gaussian with learned mean and log-variance. Appropriate for continuous real-valued data.

```java
import org.deeplearning4j.nn.conf.layers.variational.GaussianReconstructionDistribution;
import org.nd4j.linalg.activations.Activation;

// Identity activation (outputs can be any real value)
new GaussianReconstructionDistribution(Activation.IDENTITY)

// Tanh activation (outputs bounded to [-1, 1])
new GaussianReconstructionDistribution(Activation.TANH)
```

The network learns both mean and log(variance) for each output. Avoid asymmetric activations like `RELU` or `SIGMOID` as the distribution parameter space is (-inf, inf).

### BernoulliReconstructionDistribution

Models each output dimension as a Bernoulli random variable. Appropriate for binary data (pixel values 0 or 1).

```java
import org.deeplearning4j.nn.conf.layers.variational.BernoulliReconstructionDistribution;

// Uses sigmoid activation by default (outputs must be in [0, 1])
new BernoulliReconstructionDistribution()
```

The decoder output is passed through a sigmoid to produce probabilities. Do **not** use `RELU`, `TANH`, or other non-sigmoid activations — the output must be in [0, 1].

### ExponentialReconstructionDistribution

Models outputs using an exponential distribution. Appropriate for data in range [0, infinity), such as waiting times or count data.

```java
import org.deeplearning4j.nn.conf.layers.variational.ExponentialReconstructionDistribution;

new ExponentialReconstructionDistribution(Activation.IDENTITY)
```

The network models gamma = log(lambda), so the parameterisation is unconstrained and `IDENTITY` or `TANH` are appropriate activations.

### CompositeReconstructionDistribution

Combines multiple distributions for datasets with mixed data types (e.g., some continuous columns, some binary columns).

```java
import org.deeplearning4j.nn.conf.layers.variational.CompositeReconstructionDistribution;

CompositeReconstructionDistribution dist = new CompositeReconstructionDistribution.Builder()
    // First 100 output values modelled as Gaussian (continuous features)
    .addDistribution(100, new GaussianReconstructionDistribution(Activation.IDENTITY))
    // Next 50 output values modelled as Bernoulli (binary features)
    .addDistribution(50, new BernoulliReconstructionDistribution())
    .build();
```

Distributions are applied to contiguous slices of the output in the order they are added.

### LossFunctionWrapper

Allows using a standard loss function (e.g., MSE) in place of a probabilistic reconstruction distribution. This is not standard VAE design but is valid when a probabilistic interpretation is not required.

```java
import org.deeplearning4j.nn.conf.layers.variational.LossFunctionWrapper;
import org.nd4j.linalg.activations.Activation;
import org.nd4j.linalg.lossfunctions.LossFunctions;

new LossFunctionWrapper(Activation.SIGMOID,
    new org.nd4j.linalg.lossfunctions.impl.LossMSE())
```

Note: reconstruction log-probability cannot be computed when using `LossFunctionWrapper`.

---

## Training Patterns

### Pretraining (Unsupervised)

Call `pretrain(iterator)` to train using the VAE's generative objective (ELBO maximisation):

```java
DataSetIterator trainIter = /* your iterator */;
model.pretrain(trainIter);
```

During pretraining only the VAE layer parameters are updated. Add additional layers after the VAE for downstream classification or regression tasks.

### Reconstruction and Generation

After training, use the underlying `org.deeplearning4j.nn.layers.variational.VariationalAutoencoderParamInitializer` API via the layer itself:

```java
import org.deeplearning4j.nn.layers.variational.VariationalAutoencoder;

// Obtain the VAE layer from a trained MultiLayerNetwork
VariationalAutoencoder vaeLayer =
    (VariationalAutoencoder) model.getLayer(0);

// Encode: get latent mean for given input
INDArray input = /* your data, shape [batchSize, inputDim] */;
INDArray latentMean = vaeLayer.activate(input, false, LayerWorkspaceMgr.noWorkspaces());

// Reconstruct: decode a latent code back to data space
INDArray reconstructed = vaeLayer.generateAtMeanGivenZ(latentMean);

// Sample: generate new examples from the prior p(z) = N(0, I)
INDArray noise = Nd4j.randn(new long[]{numSamples, latentDim});
INDArray generated = vaeLayer.generateAtMeanGivenZ(noise);
```

### Fine-tuning After Pretraining

Stack a classification head on top and fine-tune with supervised training:

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .updater(new Adam(1e-4))
    .list()
    .layer(new VariationalAutoencoder.Builder()
        .nIn(inputDim).nOut(latentDim)
        .encoderLayerSizes(512, 256)
        .decoderLayerSizes(256, 512)
        .reconstructionDistribution(
            new GaussianReconstructionDistribution(Activation.IDENTITY))
        .build())
    .layer(new DenseLayer.Builder()
        .nIn(latentDim).nOut(128)
        .activation(Activation.RELU)
        .build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(128).nOut(numClasses)
        .activation(Activation.SOFTMAX)
        .build())
    .build();

MultiLayerNetwork model = new MultiLayerNetwork(conf);
model.init();

// Step 1: pretrain
model.pretrain(trainIter);

// Step 2: finetune (supervised)
model.fit(labelledTrainIter);
```

### Choosing `numSamples`

The `numSamples` parameter controls how many latent samples are drawn per data point during pretraining. Kingma & Welling note that `numSamples = 1` is sufficient when the minibatch size is large (e.g., >= 100). Increasing `numSamples` reduces variance in the gradient estimate but increases computation cost proportionally.

```java
.numSamples(1)   // default; appropriate for batch sizes >= 100
.numSamples(5)   // more stable gradients for small batches
```

---

## API Reference

| Class | Package |
|---|---|
| `AutoEncoder` | `org.deeplearning4j.nn.conf.layers` |
| `VariationalAutoencoder` (config) | `org.deeplearning4j.nn.conf.layers.variational` |
| `VariationalAutoencoder` (layer) | `org.deeplearning4j.nn.layers.variational` |
| `GaussianReconstructionDistribution` | `org.deeplearning4j.nn.conf.layers.variational` |
| `BernoulliReconstructionDistribution` | `org.deeplearning4j.nn.conf.layers.variational` |
| `ExponentialReconstructionDistribution` | `org.deeplearning4j.nn.conf.layers.variational` |
| `CompositeReconstructionDistribution` | `org.deeplearning4j.nn.conf.layers.variational` |
| `LossFunctionWrapper` | `org.deeplearning4j.nn.conf.layers.variational` |
| `ReconstructionDistribution` (interface) | `org.deeplearning4j.nn.conf.layers.variational` |
