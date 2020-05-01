---
description: 在 DL4J中使用NVIDIA cuDNN 库
---

# cuDNN

### 与cuDNN一起使用DL4J

DL4J支持CUDA，但可以进一步通过cuDNN加速。大多数2D 卷积神经网络层（如ConvolutionLayer、SubsamplingLayer等），以及LSTM和BatchNormalization层都支持cuDNN。

 要让DL4J加载cuDNN，我们只需要添加一个依赖项`deeplearning4j-cuda-9.2`, `deeplearning4j-cuda-10.0`, 或 `deeplearning4j-cuda-10.1`, 例如 :

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-9.2</artifactId>
    <version>1.0.0-beta6</version>
</dependency>
```

或

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-10.0</artifactId>
    <version>1.0.0-beta6</version>
</dependency>
```

或

```markup
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-cuda-10.1</artifactId>
    <version>1.0.0-beta6</version>
</dependency>
```

cuDNN的实际库没有捆绑，因此请确保从NVIDIA下载并安装适合你的平台的包

* [NVIDIA cuDNN](https://developer.nvidia.com/cudnn)

注意cuDNN和CUDA有多个组合被支持。当前，DL4J的支持下面的组合：

| CUDA 版本 | cuDNN 版本 |
| :--- | :--- |
| 9.2 | 7.2 |
| 10.0 | 7.4 |
| 10.1 | 7.6 |
| 10.2 | 7.6 |

要安装，只需将库提取到本地库使用的系统路径中找到的目录即可。最简单的方法是将它放在默认目录中的CUDA之外的其他库中。 \(`/usr/local/cuda/lib64/` 在 Linux上, `/usr/local/cuda/lib/` 在 Mac OS X, 并且 `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.2\bin\`, `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.0\bin\`, 或  `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.1\bin\` 在 Windows上\)。

或者，对于CUDA 9.2，cuDNN与[用于CUDA的JavaCPP Presets](https://github.com/bytedeco/javacpp-presets/tree/master/cuda) 的“redist”包捆绑在一起。在[同意许可](https://github.com/bytedeco/javacpp-presets/tree/master/cuda#license-agreements)之后，我们可以添加以下依赖项，来取代安装CUDA和cuDNN：

```markup
<dependency>
     <groupId>org.bytedeco</groupId>
     <artifactId>cuda-platform-redist</artifactId>
     <version>10.2-7.6-1.5.2</version>
 </dependency>
```

还要注意，默认情况下，DL4j将使用根据cuDNN可用的最快算法，但是内存使用可能溢出，导致奇怪的启动错误。当这种情况发生时，尝试通过使用[通过网络配置设置的NO\_WORKSPACE模式](https://deeplearning4j.org/api/1.0.0-beta2/org/deeplearning4j/nn/conf/layers/ConvolutionLayer.Builder.html#cudnnAlgoMode-org.deeplearning4j.nn.conf.layers.ConvolutionLayer.AlgoMode-)来减少内存使用， 替换默认的`ConvolutionLayer.AlgoMode.PREFER_FASTEST`，例如：

```java
    // 对于整个网络
    new NeuralNetConfiguration.Builder()
            .cudnnAlgoMode(ConvolutionLayer.AlgoMode.NO_WORKSPACE)
            // ...
    // 或对于单个层
    new ConvolutionLayer.Builder(h, w)
            .cudnnAlgoMode(ConvolutionLayer.AlgoMode.NO_WORKSPACE)
            // ...
```

