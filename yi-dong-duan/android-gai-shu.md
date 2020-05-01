---
description: 在Android应用中使用深度学习和神经网络
---

# Android概述

## 在Android应用中使用深度学习和神经网络

内容

* [先决条件](android-gai-shu.md#xian-jue-tiao-jian)
* [配置你的Android Studio项目](android-gai-shu.md#pei-zhi-ni-de-android-studio-xiang-mu)
* [启动异步任务](android-gai-shu.md#qi-dong-yi-bu-ren-wu)
* [创建一个神经网络](android-gai-shu.md#chuang-jian-yi-ge-shen-jing-wang-luo)
* [创建训练数据](android-gai-shu.md#chuang-jian-xun-lian-shu-ju)
* [结论](android-gai-shu.md#jie-lun)

一般来说，训练一个神经网络是一项最适合有多个GPU的强大计算机的任务。但如果你想在你简陋的安卓手机或平板电脑上做呢？好吧，这绝对是可能的。然而，考虑到一个普通的Android设备的规格，它很可能会相当慢。如果这对你来说不是问题，继续读下去。

 在本教程中，我将向您展示如何使用 [Deeplearning4J](../kai-shi/kuai-su-ru-men.md)，一个流行的基于Java的深度学习库，在Android设备上创建和训练神经网络。

## [先决条件](android-gai-shu.md)

为了获得最佳效果，您需要以下各项：

* 运行API级别21或更高的安卓设备或模拟器，内部存储空间大约为200 MB。我强烈建议您首先使用模拟器，因为您可以快速调整它，以防内存或存储空间不足。
* Android Studio 2.2 或更新版本
* 这里可以找到在Android应用程序中使用DL4J的更深入的研究。本指南涵盖依赖项、内存管理、保存设备训练模型以及在应用程序中加载预先训练的模型。

## [配置你的Android Studio项目](android-gai-shu.md)

要在项目中使用Deeplearning4J，请将以下实现依赖项添加到应用程序模块的build.gradle文件中：

```groovy
implementation (group: 'org.deeplearning4j', name: 'deeplearning4j-core', version: '{{page.version}}') {
    exclude group: 'org.bytedeco', module: 'opencv-platform'
    exclude group: 'org.bytedeco', module: 'leptonica-platform'
    exclude group: 'org.bytedeco', module: 'hdf5-platform'
}
implementation group: 'org.nd4j', name: 'nd4j-native', version: '{{page.version}}'
implementation group: 'org.nd4j', name: 'nd4j-native', version: '{{page.version}}', classifier: "android-arm"
implementation group: 'org.nd4j', name: 'nd4j-native', version: '{{page.version}}', classifier: "android-arm64"
implementation group: 'org.nd4j', name: 'nd4j-native', version: '{{page.version}}', classifier: "android-x86"
implementation group: 'org.nd4j', name: 'nd4j-native', version: '{{page.version}}', classifier: "android-x86_64"
implementation group: 'org.bytedeco', name: 'openblas', version: '0.3.7-1.5.2'
implementation group: 'org.bytedeco', name: 'openblas', version: '0.3.7-1.5.2', classifier: "android-arm"
implementation group: 'org.bytedeco', name: 'openblas', version: '0.3.7-1.5.2', classifier: "android-arm64"
implementation group: 'org.bytedeco', name: 'openblas', version: '0.3.7-1.5.2', classifier: "android-x86"
implementation group: 'org.bytedeco', name: 'openblas', version: '0.3.7-1.5.2', classifier: "android-x86_64"
implementation group: 'org.bytedeco', name: 'opencv', version: '4.1.2-1.5.2'
implementation group: 'org.bytedeco', name: 'opencv', version: '4.1.2-1.5.2', classifier: "android-arm"
implementation group: 'org.bytedeco', name: 'opencv', version: '4.1.2-1.5.2', classifier: "android-arm64"
implementation group: 'org.bytedeco', name: 'opencv', version: '4.1.2-1.5.2', classifier: "android-x86"
implementation group: 'org.bytedeco', name: 'opencv', version: '4.1.2-1.5.2', classifier: "android-x86_64"
implementation group: 'org.bytedeco', name: 'leptonica', version: '1.78.0-1.5.2'
implementation group: 'org.bytedeco', name: 'leptonica', version: '1.78.0-1.5.2', classifier: "android-arm"
implementation group: 'org.bytedeco', name: 'leptonica', version: '1.78.0-1.5.2', classifier: "android-arm64"
implementation group: 'org.bytedeco', name: 'leptonica', version: '1.78.0-1.5.2', classifier: "android-x86"
implementation group: 'org.bytedeco', name: 'leptonica', version: '1.78.0-1.5.2', classifier: "android-x86_64"
```

如果选择将依赖项的快照版本与gradle一起使用，则需要在根目录中创建pom.xml文件，并从终端对其运行 `mvn -U compile` 。您还需要在build.gradle文件的`repository{}`块中包含`mavenLocal()`。下面提供了一个pom.xml文件示例。

```markup
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>snapshots</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <dependencies>
       <dependency>
            <groupId>org.nd4j</groupId>
            <artifactId>nd4j-native-platform</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.deeplearning4j</groupId>
            <artifactId>deeplearning4j-core</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
    <repositories>
        <repository>
            <id>sonatype-nexus-snapshots</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
        </repository>
    </repositories>
</project>
```

Android Studio 3.0引入了新的Gradle，现在也应该定义annotationProcessors如果您正在使用它，请向Gradle依赖项添加以下代码：

```java
NeuralNetConfiguration.Builder nncBuilder = new NeuralNetConfiguration.Builder();
nncBuilder.updater(Updater.ADAM);
```

如您所见，DL4J依赖于ND4J，Java的N维缩写，它是一个提供快速N维数组的库。ND4J在内部依赖于一个名为OpenBLAS的库，该库包含特定于平台的本地代码。因此，您必须加载与您的Android设备架构相匹配的OpenBLAS和ND4J版本。 

DL4J和ND4J的依赖项有几个同名的文件。为了避免构建错误，请将以下排除参数添加到packagingOptions中。

```groovy
packagingOptions {
    exclude 'META-INF/DEPENDENCIES'
    exclude 'META-INF/DEPENDENCIES.txt'
    exclude 'META-INF/LICENSE'
    exclude 'META-INF/LICENSE.txt'
    exclude 'META-INF/license.txt'
    exclude 'META-INF/NOTICE'
    exclude 'META-INF/NOTICE.txt'
    exclude 'META-INF/notice.txt'
    exclude 'META-INF/INDEX.LIST'
}
```

编译后的代码将有超过65536个方法。要处理此情况，请在defaultConfig中添加以下选项：

```groovy
multiDexEnabled true
```

现在，按Sync now更新项目。最后，确保APK不同时包含lib/armeabi和lib/armeabi-v7a子目录。如果是，请将所有文件移动到其中一个或另一个，因为某些Android设备将同时存在这两个文件。

## [启动异步任务](android-gai-shu.md)

训练神经网络是CPU密集型的，这就是为什么您不想在应用程序的UI线程中进行训练。我不太确定DL4J是否在默认情况下异步训练其网络。为了安全起见，我现在将使用AsyncTask类生成一个单独的线程。

```java
AsyncTask.execute(new Runnable() {
    @Override
    public void run() {
        createAndUseNetwork();
    }
});
```

因为createAndUseNetwork\(\)方法还不存在，所以创建它。

```java
private void createAndUseNetwork() {
}
```

## [创建一个神经网络](android-gai-shu.md)

DL4J有一个非常直观的API。现在让我们用它来创建一个简单的多层感知器与隐藏层。它将获取两个输入值，并输出一个输出值。要创建层，我们将使用DenseLayer和OutputLayer类。相应地，将以下代码添加到在上一步中创建的createAndUseNetwork\(\)方法中：

```java
DenseLayer inputLayer = new DenseLayer.Builder()
        .nIn(2)
        .nOut(3)
        .name("Input")
        .build();
DenseLayer hiddenLayer = new DenseLayer.Builder()
        .nIn(3)
        .nOut(2)
        .name("Hidden")
        .build();
OutputLayer outputLayer = new OutputLayer.Builder()
        .nIn(2)
        .nOut(1)
        .name("Output")
        .build();
```

现在我们的层已经准备好了，让我们创建一个NeuralNetConfiguration.Builder对象来配置我们的神经网络。

```java
NeuralNetConfiguration.Builder nncBuilder = new NeuralNetConfiguration.Builder();
nncBuilder.updater(Updater.ADAM);
```

我们现在必须创建一个NeuralNetConfiguration.ListBuilder对象来实际连接我们的层并指定它们的顺序。

```java
NeuralNetConfiguration.ListBuilder listBuilder = nncBuilder.list();
listBuilder.layer(0, inputLayer);
listBuilder.layer(1, hiddenLayer);
listBuilder.layer(2, outputLayer);
```

另外，通过添加以下代码启用反向传播：

```java
listBuilder.backprop(true);
```

此时，我们可以将神经网络生成并初始化为多层网络类的实例。

```java
MultiLayerNetwork myNetwork = new MultiLayerNetwork(listBuilder.build());
myNetwork.init();
```

## [创建训练数据](android-gai-shu.md)

为了创建我们的训练数据，我们将使用ND4J提供的INDArray类

```text
INPUTS      EXPECTED OUTPUTS
------      ----------------
0,0         0
0,1         1
1,0         1
1,1         0
```

正如你可能已经猜到的，我们的神经网络将表现得像一个异或门。训练数据有四个样本，您必须在代码中提到它。

```java
final int NUM_SAMPLES = 4;
```

现在，为输入和预期输出创建两个INDArray对象，并用零初始化它们。

```java
INDArray trainingInputs = Nd4j.zeros(NUM_SAMPLES, inputLayer.getNIn());
INDArray trainingOutputs = Nd4j.zeros(NUM_SAMPLES, outputLayer.getNOut());
```

注意，输入数组中的列数等于输入层中的神经元数。类似地，输出数组中的列数等于输出层中的神经元数。 

用训练数据填充这些数组很容易。只需使用putScalar\(\)方法：

```java
// If 0,0 show 0
trainingInputs.putScalar(new int[]{0, 0}, 0);
trainingInputs.putScalar(new int[]{0, 1}, 0);
trainingOutputs.putScalar(new int[]{0, 0}, 0);
// If 0,1 show 1
trainingInputs.putScalar(new int[]{1, 0}, 0);
trainingInputs.putScalar(new int[]{1, 1}, 1);
trainingOutputs.putScalar(new int[]{1, 0}, 1);
// If 1,0 show 1
trainingInputs.putScalar(new int[]{2, 0}, 1);
trainingInputs.putScalar(new int[]{2, 1}, 0);
trainingOutputs.putScalar(new int[]{2, 0}, 1);
// If 1,1 show 0
trainingInputs.putScalar(new int[]{3, 0}, 1);
trainingInputs.putScalar(new int[]{3, 1}, 1);
trainingOutputs.putScalar(new int[]{3, 0}, 0);
```

我们不会直接使用INDArray对象。相反，我们将把它们转换成一个DataSet。

```java
DataSet myData = new DataSet(trainingInputs, trainingOutputs);
```

此时，我们可以通过调用神经网络的`fit()`方法并将数据集传递给它来开始训练。`for`循环控制通过网络的数据集的迭代。在本例中，它被设置为1000次迭代。

```java
for(int l=0; l<=1000; l++) {
    myNetwork.fit(myData);
}
```

就这些。你的神经网络已经可以使用了。

## [结论](android-gai-shu.md)

在本教程中，您看到了在Android Studio项目中使用Deeplearning4J库创建和训练神经网络是多么容易。不过，我想提醒你的是，在低功耗、电池供电的设备上训练神经网络可能并不总是一个好主意。

第二个例子DL4J Android应用程序包括一个用户界面可以在这里找到。这个例子使用Anderson的iris数据集在设备上训练一个神经网络，用于iris类型分类。该应用程序包括用户输入的测量值，并返回这些测量值属于三种iris类型（_Iris serosa, Iris versicolor,_ 和 _Iris virginica_）之一的概率。 

移动设备处理能力和电池寿命的限制使得训练健壮、多层网络不可行。作为在设备上训练网络的替代方法，应用程序使用的神经网络可以在桌面机上训练，通过ModelSerializer保存，然后作为预先训练的模型加载到应用程序中。第三个例子DL4J Android应用程序可以在这里找到，它加载一个预先训练的MNIST网络，并使用它对用户绘制的数字进行分类。

