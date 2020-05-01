---
description: 使用每日版本访问最新的Eclipse Deeplearning4J功能。
---

# 快照

### 内容

* [快照介绍](kuai-zhao.md#gai-shu-jie-shao)
* [安装说明](kuai-zhao.md#an-zhuang-shuo-ming)
* [局限](kuai-zhao.md#ju-xian)
* ND4J后端配置
* [Gradle用户需知](kuai-zhao.md#gradle-yong-hu-xu-zhi)

## 概述／介绍

我们提供仓库的自动化日常构建，如ND4J, DataVec, DeepLearning4j, RL4J等等，所以所有最新的功能和最新的bug修复都是每天发布的。

快照像任何其他的Maven依赖一样工作。唯一的区别是，它们是从自定义存储库提供的，而不是从Maven Central提供的。

由于正在进行的开发，快照版应该被认为比发布版更不稳定：在正常开发过程中，原则上可以在任何时候引入破坏性的变化或错误。通常情况下，应在可能的情况下使用发布版本（不是快照），除非需要修复错误或需要新的功能。

## 安装说明

**步骤1:** 若要在项目中使用快照，应将像这样将快照仓库信息添加到你的POM.xml文件中：

```markup
<repositories>
    <repository>
        <id>snapshots-repo</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>daily</updatePolicy>  <!-- Optional, update daily -->
        </snapshots>
    </repository>
</repositories>
```

**步骤2:** 请确保指定快照版本。我们遵循一个简单的规则：如果最新的稳定发布版本是`A.B.C`，快照版本将是`A.B.(C+1)-SNAPSHOT`。当前快照版本为 `1.0.0-SNAPSHOT`。有关pom.xml文件的存储库部分的更多细节，请参见[Maven ](https://maven.apache.org/settings.html#Repositories)文档

如果使用类似DL4J示例的属性，则更改：从版本：

```markup
<dl4j.version>1.0.0-beta6</dl4j.version>
<nd4j.version>1.0.0-beta6</nd4j.version>
```

改为版本:

```markup
<dl4j.version>1.0.0-SNAPSHOT</dl4j.version>
<nd4j.version>1.0.0-SNAPSHOT</nd4j.version>
```

**使用快照的示例pom.xml**

这里提供了示例pom.xml：[使用快照的示例pom.xml](https://gist.github.com/AlexDBlack/28b0c9a72bce562c8782be326a6e2aaa)。这是从DL4J独立示例项目中获取的，并且使用上面的步骤1和2进行了修改。原件（使用最新版本）可以在[这里](https://github.com/deeplearning4j/dl4j-examples/blob/master/standalone-sample-project/pom.xml)找到

## 局限

`-platform`（所有操作系统）和单操作系统（非平台）快照依赖项都已释放。由于快照的多平台构建特性，`-platform`工件可能（虽然很少）暂时不同步，这可能会导致构建问题。 

如果您只在一个平台上构建和部署，那么使用如下non-platform构件会比较安全

```markup
        <dependency>
            <groupId>org.nd4j</groupId>
            <artifactId>nd4j-native</artifactId>
            <version>${nd4j.version}</version>
        </dependency>
```

## 用于快照的有用的Maven命令

在Maven中使用快照依赖时可能有用的两个命令如下：

1. `-U` - 例如, 在  `mvn package -U`. 这个 `-U` 选项强制Maven去检查 \(并且有必要的话，下载\)最新的快照版。这在你需要确保你有绝对最新快照版时很有用。 
2. `-nsu` - 例如, 在 `mvn package -nsu`. 这个 `-nsu` 选项停止Maven正在进行的快照版检查。但是，请注意，只有当你有一些快照依赖项已经下载到本地Maven缓存 \(.m2 目录\)中时，你的构建才会成功使用此选项。

另一种方法（1）是在`<repositories>节点`设置 `<updatePolicy>always</updatePolicy>`

另一种方法（2）是在`<repositories>节点`设置设置`<updatePolicy>never</updatePolicy>`

## Gradle用户需知

快照不能与Gradle一起工作。你必须使用Maven下载文件。之后，你可以尝试使用 `mavenLocal()`使用本地Maven仓库。

最小文件像这样：

```text
version '1.0-SNAPSHOT'

apply plugin: 'java'

sourceCompatibility = 1.8

repositories {
    maven { url "https://oss.sonatype.org/content/repositories/snapshots" }
    mavenCentral()
}

dependencies {
    compile group: 'org.deeplearning4j', name: 'deeplearning4j-core', version: '1.0.0-SNAPSHOT'
    compile group: 'org.deeplearning4j', name: 'deeplearning4j-modelimport', version: '1.0.0-SNAPSHOT'
    compile "org.nd4j:nd4j-native:1.0.0-SNAPSHOT"
    // Use windows-x86_64 or linux-x86_64 if you are not on macos
    compile "org.nd4j:nd4j-native:1.0.0-SNAPSHOT:macosx-x86_64"
    testCompile group: 'junit', name: 'junit', version: '4.12'

}
```

应该在理论上工作，但事实并非如此。这是因为[Gradle的一个缺陷](https://github.com/gradle/gradle/issues/2882)。带有快照的Gradle和Maven分类器似乎有问题。

值得注意的是，当使用Gradle上的ND4J本地后端（和SBT -但不是Maven）时，需要添加OpenBLAS作为依赖项。我们在-platform pom上为你做了这件事。参考-platform pom [这里](https://github.com/deeplearning4j/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-backend-impls/nd4j-native-platform/pom.xml#L19)来重复检查你的依赖关系。请注意，这些是版本属性。有关运行 nd4j-native所需的OpenBLAS和javacpp presets当前版本，请参阅POM的`<properties>`节点。

