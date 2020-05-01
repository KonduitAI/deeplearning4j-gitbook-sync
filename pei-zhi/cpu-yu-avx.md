---
description: ND4J/Deeplearning4j中的 CPU与AVX支持
---

# CPU 与 AVX

## 什么是AVX，为什么重要？

AVX（高级向量扩展）是一组用于加速数值计算的CPU指令。更多细节见[维基百科](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions)。

请注意，AVX仅适用于x86设备的nd4j-native（CPU）后端，而不适用于GPU和ARM/PPC设备。

AVX的重要性：性能。您希望使用系统支持的最高级别AVX编译的ND4J版本。

不同CPU的AVX支持-摘要：

* 大多数现代x86 CPU：支持AVX2
* 一些高端服务器CPU:AVX512可能被支持
* 旧CPU（2012年之前）和低功耗x86（Atom、Celeron）：不支持AVX（通常）

注意，支持更高版本AVX的CPU还包括所有更早版本。这意味着可以在支持AVX512的系统上运行通用X86或AVX2二进制指令。但是，不可能在不支持这些指令的CPU上运行为更高版本（如AVX512）构建的二进制指令。

在版本1.0.0-beta6和更高版本中，如果未优化配置AVX，则可能会收到以下警告：

```text
*********************************** CPU Feature Check Warning ***********************************
Warning: Initializing ND4J with Generic x86 binary on a CPU with AVX/AVX2 support
Using ND4J with AVX/AVX2 will improve performance. See deeplearning4j.org/cpu for more details
Or set environment variable ND4J_IGNORE_AVX=true to suppress this warning
************************************************************************************************
```

## 在ND4J/DL4J中配置AVX

如前所述，为了获得最佳性能，您应该使用与CPU支持的AVX级别相匹配的ND4J版本。

对于nd4j/nd4j-platform依赖项，ND4J默认配置（仅包括nd4j-native 或 nd4j-native-platform依赖项而不包括maven分类器配置）是“generic x86”（无AVX）。

要配置AVX2和AVX512，需要为适当的架构指定一个分类器。

为X86体系结构提供了以下二进制文件（nd4j-native分类器）：

* Generic x86 \(no AVX\): `linux-x86_64`, `windows-x86_64`, `macosx-x86_64` 
* AVX2: `linux-x86_64-avx2`, `windows-x86_64-avx2`, `macosx-x86_64-avx2`
* AVX512: `linux-x86_64-avx512`

**示例：在Windows上配置AVX2（Maven pom.xml）**

```markup
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
</dependency>

<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
    <classifier>windows-x86_64-avx2</classifier>
</dependency>
```

**示例：在Linux上配置AVX512（Maven pom.xml）**

```markup
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
</dependency>

<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native</artifactId>
    <version>${nd4j.version}</version>
    <classifier>linux-x86_64-avx512</classifier>
</dependency>
```

请注意，您需要nd4j-native依赖项-有或没有分类器。

在上面的例子中，假设Maven属性 `nd4j.version` 设置为适当的ND4J版本， `1.0.0-beta6` 

