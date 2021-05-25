---
title: Using Deeplearning4j with cuDNN
short_title: cuDNN
description: Using the NVIDIA cuDNN library with DL4J.
category: Configuration
weight: 3
---

# cuDNN

## Using Deeplearning4j with cuDNN


There are 2 ways of using cudnn with deeplearning4j. One is an older way described below that is built in to the various deep learning4j layers at the java level.

The other is to use the new nd4j cuda bindings that link to cudnn at the c++ level. Both will be described below. The newer way first, followed by the old way.


## Cudnn setup

The actual library for cuDNN is not bundled, so be sure to download and install the appropriate package for your platform from NVIDIA:

* [NVIDIA cuDNN](https://developer.nvidia.com/cudnn)

Note there are multiple combinations of cuDNN and CUDA supported.  Deeplearning4j's cuda support is based on [javacpp's cuda bindings](https://search.maven.org/artifact/org.bytedeco/cuda).
The way to read the versioning is: cuda version - cudnn version - javacpp version.
For example, if the cuda version is set to 11.2,  you can expect us to support cudnn 8.1.



To install, simply extract the library to a directory found in the system path used by native libraries. The easiest way is to place it alongside other libraries from CUDA in the default directory \(`/usr/local/cuda/lib64/` on Linux, `/usr/local/cuda/lib/` on Mac OS X, and `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.0\bin\`, `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.1\bin\`, or `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.2\bin\` on Windows\).

Alternatively, in the case of the most recent supported cuda version, cuDNN comes bundled with the "redist" package of the [JavaCPP Presets for CUDA](https://github.com/bytedeco/javacpp-presets/tree/master/cuda). [After agreeing to the license](https://github.com/bytedeco/javacpp-presets/tree/master/cuda#license-agreements), we can add the following dependencies instead of installing CUDA and cuDNN:

```markup
 <dependency>
     <groupId>org.bytedeco</groupId>
     <artifactId>cuda-platform-redist</artifactId>
     <version>$CUDA_VERSION-$CUDNN_VERSIUON-$JAVACPP_VERSION</version>
 </dependency>
```
The same versioning scheme for redist applies to the cuda bindings that leverage an installed cuda.




## Using cuDNN via nd4j

Similar to our avx bindings, nd4j leverages our c++ library libnd4j for running mathematical operations. In order to use cudnn, all you need to do is change the cuda backend dependency from:

```markup
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.2</artifactId>
    <version>1.0.0-M1</version>
</dependency>
```

or for cuda 11.0:
```markup
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.0</artifactId>
    <version>1.0.0-M1</version>
</dependency>
```

to


```markup
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.2</artifactId>
    <version>1.0.0-M1</version>
</dependency>
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.2</artifactId>
    <version>1.0.0-M1</version>
    <classifier>linux-x86_64-cudnn</classifier>
</dependency>

```

or for cuda 11.0:
```markup
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.2</artifactId>
    <version>1.0.0-M1</version>
</dependency>
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-11.2</artifactId>
    <version>1.0.0-M1</version>
    <classifier>linux-x86_64-cudnn</classifier>
</dependency>
```

Note that we are only adding an additional dependency. The reason we use an additional classifier is to pull in an optional dependency on cudnn based routines.
The default does not use cudnn, but instead built in standalone routines for various operations implemented in cudnn such as conv2d and lstm.

For users of the -platform dependencies such as nd4j-cuda-11.2-platform, this classifier is still required. The -platform dependencies try to set sane defaults for each platform, but give users
the option to include whatever they want. If you need optimizations, please become familiar with this.




## Using cudnn via deeplearning4j

Deeplearning4j supports CUDA but can be further accelerated with cuDNN. Most 2D CNN layers \(such as ConvolutionLayer, SubsamplingLayer, etc\), and also LSTM and BatchNormalization layers support CuDNN.

The only thing we need to do to have DL4J load cuDNN is to add a dependency on `deeplearning4j-cuda-11.0`, or `deeplearning4j-cuda-11.2`, for example:

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-11.0</artifactId>
    <version>1.0.0-M1</version>
</dependency>
```

or

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-11.2</artifactId>
    <version>1.0.0-M1</version>
</dependency>
```

or

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-11.2</artifactId>
    <version>1.0.0-M1</version>
</dependency>
```



Also note that, by default, Deeplearning4j will use the fastest algorithms available according to cuDNN, but memory usage may be excessive, causing strange launch errors. When this happens, try to reduce memory usage by using the [`NO_WORKSPACE` mode settable via the network configuration](/api/%7B%7Bpage.version%7D%7D/org/deeplearning4j/nn/conf/layers/ConvolutionLayer.Builder.html#cudnnAlgoMode-org.deeplearning4j.nn.conf.layers.ConvolutionLayer.AlgoMode-), instead of the default of `ConvolutionLayer.AlgoMode.PREFER_FASTEST`, for example:

```java
    // for the whole network
    new NeuralNetConfiguration.Builder()
            .cudnnAlgoMode(ConvolutionLayer.AlgoMode.NO_WORKSPACE)
            // ...
    // or separately for each layer
    new ConvolutionLayer.Builder(h, w)
            .cudnnAlgoMode(ConvolutionLayer.AlgoMode.NO_WORKSPACE)
            // ...
```

