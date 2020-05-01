---
description: Eclipse Deeplearning4J的硬件设置，包括GPU和CUDA。
---

# GPU/CPU设置

### 用于GPU和CPU的ND4J后端

你可以通过更改ND4J的POM.xml文件中的依赖项来为后端线性代数操作选择GPU或本地CPU。你的选择将影响应用程序中正在使用的ND4J和DL4J。

如果你的CUDA v9.2+已经安装并且有NVIDIA兼容的硬件，那么你的依赖声明将看起来像：

```markup
<dependency>
 <groupId>org.nd4j</groupId>
 <artifactId>nd4j-cuda-9.2</artifactId>
 <version>1.0.0-beta2</version>
</dependency>
```

否则，你将需要使用ND4J的本地实现作为CPU后端：

```markup
<dependency>
 <groupId>org.nd4j</groupId>
 <artifactId>nd4j-native</artifactId>
 <version>1.0.0-beta2</version>
</dependency>
```

### 系统架构

如果你在多个操作系统/系统架构上开发项目，则可以在`artifactId`的末尾添加`-platform`，该`artifactId`将下载大多数主要系统的二进制文件。

```markup
<dependency>
 ...
 <artifactId>nd4j-native-platform</artifactId>
 ...
</dependency>
```

### 多GPU

如果你有多个GPU但你的系统迫使你 只能用一个，你可以用 helper `CudaEnvironment.getInstance().getConfiguration().allowMultiGPU(true);`作为你的 `main()方法的第一行`。

### CuDNN

查看我们的 [CuDNN](https://deeplearning4j.org/docs/latest/deeplearning4j-config-cudnn) 页。

### CUDA 安装

在NVIDIA [网站](http://docs.nvidia.com/cuda/) 上查看CUDA安装说明

