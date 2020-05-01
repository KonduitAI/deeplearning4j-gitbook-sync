---
description: 为Deeplearning4j配置Maven构建工具。
---

# Maven

### 配置 Maven 构建工具

你可以通过Maven使用DL4J，将以下内容添加到你的`pom.xml`：

```markup
<dependencies>
  <dependency>
      <groupId>org.deeplearning4j</groupId>
      <artifactId>deeplearning4j-core</artifactId>
      <version>1.0.0-beta6</version>
  </dependency>
</dependencies>
```

下面的指令适用于所有DL4J和ND4J子模块，例如 `deeplearning4j-api`, `deeplearning4j-scaleout`和ND4J后端。

### 添加一个后端

DL4J依赖于ND4J用于硬件特定的实现和张量操作。通过将下面的代码粘贴到你的`pom.xml文件`中来添加后端:

```markup
<dependencies>
  <dependency>
      <groupId>org.nd4j</groupId>
      <artifactId>nd4j-native-platform</artifactId>
      <version>1.0.0-beta6</version>
  </dependency>
</dependencies>
```

你还可以把 [GPUs](https://deeplearning4j.org/docs/latest/deeplearning4j-config-gpu-cpu) 替换为标准的CPU实现

