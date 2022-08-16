---
description: Simple and sequential network configuration.
---

# Multi Layer Network

The `MultiLayerNetwork` class is the simplest network configuration API available in Eclipse Deeplearning4j. This class is useful for beginners or users who do not need a complex and branched network graph.

You will not want to use `MultiLayerNetwork` configuration if you are creating complex loss functions, using graph vertices, or doing advanced training such as a triplet network. This includes popular complex networks such as InceptionV4.

## Usage

The example below shows how to build a simple linear classifier using `DenseLayer` (a basic multiperceptron layer).

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(seed)
    .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
    .learningRate(learningRate)
    .updater(Updater.NESTEROVS).momentum(0.9)
    .list()
    .layer(0, new DenseLayer.Builder().nIn(numInputs).nOut(numHiddenNodes)
            .weightInit(WeightInit.XAVIER)
            .activation("relu")
            .build())
    .layer(1, new OutputLayer.Builder(LossFunction.NEGATIVELOGLIKELIHOOD)
            .weightInit(WeightInit.XAVIER)
            .activation("softmax").weightInit(WeightInit.XAVIER)
            .nIn(numHiddenNodes).nOut(numOutputs).build())
    .pretrain(false).backprop(true).build();
```

You can also create convolutional configurations:

```java
MultiLayerConfiguration.Builder builder = new NeuralNetConfiguration.Builder()
    .seed(seed)
    .regularization(true).l2(0.0005)
    .learningRate(0.01)
    .weightInit(WeightInit.XAVIER)
    .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
    .updater(Updater.NESTEROVS).momentum(0.9)
    .list()
    .layer(0, new ConvolutionLayer.Builder(5, 5)
            //nIn and nOut specify depth. nIn here is the nChannels and nOut is the number of filters to be applied
            .nIn(nChannels)
            .stride(1, 1)
            .nOut(20)
            .activation("identity")
            .build())
    .layer(1, new SubsamplingLayer.Builder(SubsamplingLayer.PoolingType.MAX)
            .kernelSize(2,2)
            .stride(2,2)
            .build())
    .layer(2, new ConvolutionLayer.Builder(5, 5)
            //Note that nIn need not be specified in later layers
            .stride(1, 1)
            .nOut(50)
            .activation("identity")
            .build())
    .layer(3, new SubsamplingLayer.Builder(SubsamplingLayer.PoolingType.MAX)
            .kernelSize(2,2)
            .stride(2,2)
            .build())
    .layer(4, new DenseLayer.Builder().activation("relu")
            .nOut(500).build())
    .layer(5, new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nOut(outputNum)
            .activation("softmax")
            .build());
```
