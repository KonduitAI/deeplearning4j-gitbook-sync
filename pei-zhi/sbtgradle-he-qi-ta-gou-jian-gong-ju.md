---
description: 为Deeplearning4j配置构建工具。
---

# SBT/Gradle和其它构建工具

### 配置你的构建工具

虽然我们鼓励Deeplearning4j、ND4J和DataVec用户使用Maven，但是为如何为其它工具配置构建文件编写文档是值得的，就像Ivy, Gradle 和 SBT，特别是对于Android项目，Google更喜欢Gradle而不是Maven。

下面的指令适用于所有DL4J和ND4J子模块，例如deeplearning4j-api、deeplearning4j-scaleout和ND4J后端。

### Gradle

你可以在Gradle上使用DL4J，通过添加如下代码到你的build.gradle的依赖块中

```text
compile "org.deeplearning4j:deeplearning4j-core:1.0.0-beta6"
```

通过添加如下代码到你的 build.gradle 来添加一个后端:

```text
compile "org.nd4j:nd4j-native-platform:1.0.0-beta6"
```

你还可以把 [GPUs](https://deeplearning4j.org/docs/latest/deeplearning4j-config-gpu-cpu) 替换为标准的CPU实现。

### SBT

你可以通过添加如下代码到build.sbt以便在SBT中使用DL4J：

```markup
libraryDependencies += "org.deeplearning4j" % "deeplearning4j-core" % "1.0.0-beta6"
```

通过添加如下代码到你的 build.sbt 来添加一个后端:

```text
libraryDependencies += "org.nd4j" % "nd4j-native-platform" % "1.0.0-beta6"
```

你还可以把 [GPUs](https://deeplearning4j.org/docs/latest/deeplearning4j-config-gpu-cpu) 替换为标准的CPU实现。

### Ivy

你可以通过添加如下代码到ivy.xml以便在ivy中使用DL4J：

```markup
<dependency org="org.deeplearning4j" name="deeplearning4j-core" rev="1.0.0-beta6" conf="build" />
```

通过添加如下代码到你的 ivy.xml 来添加一个后端:

```markup
<dependency org="org.nd4j" name="nd4j-native-platform" rev="1.0.0-beta6" conf="build" />
```

你还可以把 [GPUs](https://deeplearning4j.org/docs/latest/deeplearning4j-config-gpu-cpu) 替换为标准的CPU实现。

### Leinengen

Clojure程序员可能希望使用[Leiningen](https://github.com/technomancy/leiningen/)或 [Boot](http://boot-clj.com/)与Maven一起工作。[这里有一个Leiningen教程](https://github.com/technomancy/leiningen/blob/master/doc/TUTORIAL.md)。

注意：你仍然需要下载ND4J、DataVec和Deeplearning4j，或者双击Maven/Ivy/Gradle下载的相应JAR文件，以便在Eclipse安装中安装它们。

