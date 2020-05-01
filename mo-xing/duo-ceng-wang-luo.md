---
description: 简单和序列网络配置。
---

# 多层网络

### 为什么用多层网络?

MultiLayerNetwork类是Eclipse DL4J中可用的最简单的网络配置API。该类对于不需要复杂和分支的网络图的初学者或用户很有用。

如果你正在创建复杂的损失函数、使用图顶点或执行类似如三重网络的高级训练，则不希望使用MultiLayerNetwork配置。这包括流行的复杂网络，如InceptionV4。

### 用法

下面的例子展示了如何使用`DenseLayer（`一个基本的多感知器层）来构建一个简单的线性分类器。

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(seed)
    .iterations(1)
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

还可以创建卷积配置：

```java
MultiLayerConfiguration.Builder builder = new NeuralNetConfiguration.Builder()
    .seed(seed)
    .iterations(iterations)
    .regularization(true).l2(0.0005)
    .learningRate(0.01)//.biasLearningRate(0.02)
    //.learningRateDecayPolicy(LearningRatePolicy.Inverse).lrPolicyDecayRate(0.001).lrPolicyPower(0.75)
    .weightInit(WeightInit.XAVIER)
    .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
    .updater(Updater.NESTEROVS).momentum(0.9)
    .list()
    .layer(0, new ConvolutionLayer.Builder(5, 5)
            //nIn 与 nOut 指定深度。这里是nChannels和nOut是要应用的过滤器的数量。
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
            //注意，在稍后的层中不需要指定nIn。
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
            .build())
    .backprop(true).pretrain(false);
```

