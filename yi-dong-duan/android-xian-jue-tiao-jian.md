---
description: 为DL4J设置和配置Android Studio。
---

# Android先决条件

## Android中DL4J的先决条件和配置

内容

* [先决条件](android-xian-jue-tiao-jian.md#xian-jue-tiao-jian)
* [必需的依赖项](android-xian-jue-tiao-jian.md#bi-xu-de-yi-lai-xiang)
* [使用ProGuard管理依赖项](android-xian-jue-tiao-jian.md#shi-yong-proguard-guan-li-yi-lai-xiang)
* [内存管理](android-xian-jue-tiao-jian.md#nei-cun-guan-li)
* [在Android上保存和加载网络](android-xian-jue-tiao-jian.md#zai-android-shang-bao-cun-he-jia-zai-wang-luo)

虽然神经网络通常在使用多个GPU的功能强大的计算机上运行，但Deeplearning4J与Android平台的兼容性使得在Android应用程序中使用DL4J神经网络成为可能。本教程将介绍为构建DL4J应用程序设置Android Studio的基础知识。下面概述了为减轻低功耗移动设备的限制而需要的依赖项、内存管理和编译排除的几种配置。如果你只想让一个DL4J应用在你的设备上运行，你可以跳到一个简单的演示应用程序，它训练一个神经网络Iris花分类，在这里可用。

## [先决条件](android-xian-jue-tiao-jian.md)

* Android Studio 2.2 或者更新的，可以在[这里](https://developer.android.com/studio/index.html#Other)下载。
* Android Studio 版本2.2及更高版本带有最新的嵌入式OpenJDK；但是，建议您自己安装JDK，因为这样您就可以独立于Android Studio进行更新。Android Studio 3.0及更高版本支持所有Java 7和Java 8语言特性的子集。Java JDKs可以从Oracle的网站下载。
* 在Android studio中，Android SDK管理器可用于安装Android构建工具24.0.1或更高版本、SDK平台24或更高版本以及Android支持库。
* 运行API级别21或更高级别的Android设备或模拟器。建议至少有200 MB的内部存储空间可用。

我们还建议您下载并安装IntelliJ IDEA、Maven和完整的dl4j-examples目录，以便在桌面上而不是android studio上构建、构建和训练神经网络。

## [必需的依赖项](android-xian-jue-tiao-jian.md)

为了在Android项目中使用Deeplearning4J，您需要将以下依赖项添加到应用程序模块的build.gradle文件中。根据应用程序中使用的神经网络类型，可能需要添加其他依赖项。

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
testimplementation 'junit:junit:4.12'
```

DL4J依赖于ND4J，ND4J是一个提供快速n维数组的库。ND4J反过来依赖于一个特定于平台的本地代码库JavaCPP，因此您必须加载一个与Android设备架构相匹配的ND4J版本。可以包括-x86和-arm类型以支持多种设备处理器类型。 

上述依赖项包含多个名称相同的文件，必须使用packagingOptions的以下排除参数来处理这些文件。

```java
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

将上述依赖项和排除项添加到build.gradle文件后，请尝试将gradle同步，以查看是否需要任何其他排除项。错误消息将标识应添加到排除列表中的文件路径。带有文件路径的错误消息示例是： _&gt; More than one file was found with OS independent path 'org/bytedeco/javacpp/ windows-x86\_64/msvp120.dll_”编译这些依赖项涉及大量文件，因此有必要在defaultConfig中将multiDexEnabled设置为true。

```java
multiDexEnabled true
```

junit模块版本中的冲突通常会导致以下错误：_&gt; Conflict with dependency 'junit:junit' in project ':app'_。应用程序（4.8.2）和测试应用程序（4.12）的解析版本不同。这可以通过强制所有junit模块使用相同的版本来抑制：

```java
configurations.all {
    resolutionStrategy.force 'junit:junit:4.12'
}
```

## [使用ProGuard管理依赖项](android-xian-jue-tiao-jian.md)

DL4J依赖项编译大量文件。ProGuard可以用来最小化APK文件的大小。ProGuard从打包的应用程序（包括代码库中的应用程序）中检测并删除未使用的类、字段、方法和属性。你可以在[这里](https://developer.android.com/studio/build/shrink-code.html)学习更多关于使用ProGuard的知识。要使用ProGuard启用代码收缩，请将minifyEnabled true添加到build.gradle文件中的相应生成类型。

```java
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

建议将Android SDK中的ProGuard升级到最新版本（5.1或更高版本）。请注意，升级构建工具或SDK的其他方面可能会导致ProGuard重置为SDK附带的版本。为了强制ProGuard使用Android Gradle默认版本以外的其他版本，可以在`build.gradle`文件的buildscript中包含以下内容：

```java
buildscript {
    configurations.all {
        resolutionStrategy {
            force 'net.sf.proguard:proguard-gradle:5.3.2'
        }
    }
}
```

ProGuard优化并减少Android应用程序中的代码量，以便使if更小更快。不幸的是，ProGuard默认删除注释，包括javaCV使用的@Platform注释。要使ProGuard保留这些注释并保留本机方法，请将以下标志添加到progaurd-rules.pro文件中。

```java
# enable optimization
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
-optimizationpasses 5
-allowaccessmodification
-dontwarn org.apache.lang.**
-ignorewarnings

-keepattributes *Annotation*
# JavaCV
-keep @org.bytedeco.javacpp.annotation interface * {*;}
-keep @org.bytedeco.javacpp.annotation.Platform public class *
-keepclasseswithmembernames class * {@org.bytedeco.* <fields>;}
-keepclasseswithmembernames class * {@org.bytedeco.* <methods>;}

-keepattributes EnclosingMethod
-keep @interface org.bytedeco.javacpp.annotation.*,javax.inject.*

-keepattributes *Annotation*, Exceptions, Signature, Deprecated, SourceFile, SourceDir, LineNumberTable, LocalVariableTable, LocalVariableTypeTable, Synthetic, EnclosingMethod, RuntimeVisibleAnnotations, RuntimeInvisibleAnnotations, RuntimeVisibleParameterAnnotations, RuntimeInvisibleParameterAnnotations, AnnotationDefault, InnerClasses
-keep class org.bytedeco.javacpp.** {*;}
-dontwarn java.awt.**
-dontwarn org.bytedeco.javacv.**
-dontwarn org.bytedeco.javacpp.**
# end javacv

# This flag is needed to keep native methods
-keepclasseswithmembernames class * {
 native <methods>;
}

-keep public class * extends android.view.View {
 public <init>(android.content.Context);
 public <init>(android.content.Context, android.util.AttributeSet);
 public <init>(android.content.Context, android.util.AttributeSet, int);
 public void set*(...);
}

-keepclasseswithmembers class * {
 public <init>(android.content.Context, android.util.AttributeSet);
}

-keepclasseswithmembers class * {
 public <init>(android.content.Context, android.util.AttributeSet, int);
}

-keepclassmembers class * extends android.app.Activity {
 public void *(android.view.View);
}

# For enumeration classes
-keepclassmembers enum * {
 public static **[] values();
 public static ** valueOf(java.lang.String);
}

-keep class * implements android.os.Parcelable {
 public static final android.os.Parcelable$Creator *;
}

-keepclassmembers class **.R$* {
 public static <fields>;
}

-keep class android.support.v7.app.** { *; }
-keep interface android.support.v7.app.** { *; }
-keep class com.actionbarsherlock.** { *; }
-keep interface com.actionbarsherlock.** { *; }
-dontwarn android.support.**
-dontwarn com.google.ads.**

# Flags to keep standard classes
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgent
-keep public class * extends android.preference.Preference
-keep public class * extends android.support.v7.app.Fragment
-keep public class * extends android.support.v7.app.DialogFragment
-keep public class * extends com.actionbarsherlock.app.SherlockListFragment
-keep public class * extends com.actionbarsherlock.app.SherlockFragment
-keep public class * extends com.actionbarsherlock.app.SherlockFragmentActivity
-keep public class * extends android.app.Fragment
-keep public class com.android.vending.licensing.ILicensingService
```

测试应用程序是检查是否有由不适当删除的代码引起任何错误的最好方法；但是，您也可以通过查看保存在/build/outputs/mapping/release/中的usage.txt输出文件来检查删除的内容。 

要修复错误并强制ProGuard保留某些代码，请在ProGuard配置文件中添加-keep行。例如：

```java
-keep public class MyClass
```

## [内存管理](android-xian-jue-tiao-jian.md)

通过将android:largeHeap=“true”添加到清单文件中，增加分配给应用程序的内存也可能是有利的。分配更大的堆意味着您可以降低在内存密集型操作期间抛出OutOfMemoryError的风险。

```markup
android:largeHeap="true"
```

从0.9.0版开始，ND4J提供了一个额外的内存管理模型：工作间。工作间允许您为循环工作负载重用内存，而无需使用JVM垃圾收集器进行堆外内存跟踪。D4L工作间允许在try/catch块之前预先分配内存，并在该块内重新使用内存。

 如果训练过程使用工作区，建议在调用model.fit\(\)之前禁用或减少定期GC调用的频率。

```java
// 这将gc调用的频率限制为5000毫秒
Nd4j.getMemoryManager().setAutoGcWindow(5000)

//这将完全禁用它
Nd4j.getMemoryManager().togglePeriodicGc(false);
```

The example below illustrates the use of a Workspace for memory allocation in the AsyncTask of and Android Application. More information concerning ND4J Workspaces can be found [here](https://deeplearning4j.org/workspaces).

下面的示例演示了在Android应用程序的AsyncTask中使用工作间进行内存分配。有关ND4J工作间的更多信息可以在[这里](https://deeplearning4j.org/workspaces)找到。

```java
import org.nd4j.linalg.api.memory.MemoryWorkspace;
import org.nd4j.linalg.api.memory.conf.WorkspaceConfiguration;
import org.nd4j.linalg.api.memory.enums.AllocationPolicy;
import org.nd4j.linalg.api.memory.enums.LearningPolicy;


private class AsyncTaskRunner extends AsyncTask<String, Integer, INDArray> {

    // 在调用后台线程之前在UI中运行
    @Override
    protected void onPreExecute() {
        super.onPreExecute();
    }

    //在后台线程上运行，这是我们启动工作间的地方
    protected INDArray doInBackground(String... params) {

        //我们将创建预先分配了10MB内存空间的配置
        WorkspaceConfiguration initialConfig = WorkspaceConfiguration.builder()
                .initialSize(10 * 1024L * 1024L)
                .policyAllocation(AllocationPolicy.STRICT)
                .policyLearning(LearningPolicy.NONE)
                .build();

        INDArray result = null;

        try(MemoryWorkspace ws = Nd4j.getWorkspaceManager().getAndActivateWorkspace(initialConfig, "SOME_ID")) {
        //现在，在此try块中创建的INDArrays将从此工作间池中分配

            //加载经过训练的模型
            File file = new File(Environment.getExternalStorageDirectory() + "/trained_model.zip");
            MultiLayerNetwork restored = ModelSerializer.restoreMultiLayerNetwork(file);

            //在INDArray中创建输入
            INDArray inputData = Nd4j.zeros(1, 4);

            inputData.putScalar(new int[]{0, 0}, 1);
            inputData.putScalar(new int[]{0, 1}, 0);
            inputData.putScalar(new int[]{0, 2}, 1);
            inputData.putScalar(new int[]{0, 3}, 0);

            result = restored.output(inputData);

        }
        catch(IOException ex){Log.d("AsyncTaskRunner2 ", "catchIOException = " + ex  );}

        return result;
    }

    protected void onProgressUpdate(Integer... values) {
        super.onProgressUpdate(values);
    }

    protected void onPostExecute(INDArray result) {
        super.onPostExecute(result);
     //Handle results and update UI here.
    }

}
```

## [在Android上保存和加载网络](android-xian-jue-tiao-jian.md)

在构建运行神经网络的Android应用程序时，需要考虑性能限制。在设备上训练一个神经网络是可能的，但是应该只尝试使用层、节点和迭代次数有限的网络。第一个演示应用程序DL4JIrisClassifierDemo能够在15秒内在标准设备上进行训练。

当在设备上进行训练是一个合理的选择时，一旦初始训练完成，就可以将训练的模型保存在手机的外部存储器中，从而提高应用程序的性能。训练后的模型可以用作应用程序资源。这种方法对于训练用户输入数据的网络是有用的。下面的代码演示如何训练网络并将其保存在手机的外部资源中。

 对于API23及更高版本，您将需要在清单中包含权限，并在activity中以编程方式请求读写权限。所需的清单权限为：

```markup
<manifest>
        <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
        <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
        ...
```

您需要在activity中实现AActivityCompat.OnRequestPermissionsResultCallback，然后检查权限状态。

```java
public class MainActivity extends AppCompatActivity
        implements ActivityCompat.OnRequestPermissionsResultCallback {

    private static final int REQUEST_EXTERNAL_STORAGE = 1;
    private static String[] PERMISSIONS_STORAGE = {
            Manifest.permission.READ_EXTERNAL_STORAGE,
            Manifest.permission.WRITE_EXTERNAL_STORAGE
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        verifyStoragePermission(MainActivity.this);
    //…
    }

    public static void verifyStoragePermission(Activity activity) {
        // 获取权限状态
        int permission = ActivityCompat.checkSelfPermission(activity, Manifest.permission.WRITE_EXTERNAL_STORAGE);
        if (permission != PackageManager.PERMISSION_GRANTED) {
        // 我们没有许可我们请求它
        ActivityCompat.requestPermissions(
                    activity,
                    PERMISSIONS_STORAGE,
                    REQUEST_EXTERNAL_STORAGE
            );
        }
    }
```

要在设备上训练后保存网络，请在try catch块中使用OutputStream。

```java
try {
    File file = new File(Environment.getExternalStorageDirectory() + "/trained_model.zip");
    OutputStream outputStream = new FileOutputStream(file);
    boolean saveUpdater = true;
    ModelSerializer.writeModel(myNetwork, outputStream, saveUpdater);

} catch (Exception e) {
    Log.e("saveToExternalStorage error", e.getMessage());
}
```

要从存储中加载经过训练的网络，可以使用restoreMultiLayerNetwork方法。

```java
try{
    //加载模型
    File file = new File(Environment.getExternalStorageDirectory() + "/trained_model.zip");
    MultiLayerNetwork restored = ModelSerializer.restoreMultiLayerNetwork(file);

} catch (Exception e) {
    Log.e("Load from External Storage error", e.getMessage());
}
```

对于更大或更复杂的神经网络，如卷积神经网络或循环神经网络，在设备上进行训练是不现实的选择，因为在网络训练过程中，长时间的处理可能会产生OutOfMemoryError并导致不良的用户体验。作为替代方案，神经网络可以在桌面端训练，通过ModelSerializer保存，然后作为预先训练的模型加载到应用程序中。在Android应用程序中使用预先训练的模型可以通过以下步骤实现：

* 在桌面端训练模型并通过modelSerializer保存。
* 在应用程序的res目录中创建原始资源文件夹。
* 将model.zip文件复制到raw文件夹中。
* 使用try/catch块中的inputStream从资源访问它。

```java
try {
// 加载模型文件名(yourModel.zip).
        InputStream is = getResources().openRawResource(R.raw.yourModel);

// 加载yourModel.zip.
        MultiLayerNetwork restored = ModelSerializer.restoreMultiLayerNetwork(is);

// 使用你的模型。
        INDArray results = restored.output(input)
        System.out.println("Results: "+ results );
// 处理异常错误
} catch(IOException e) {
        e.printStackTrace();
    }
```

## 下一步：Android上的预训练DL4J模型

这里可以找到使用预训练模型的示例应用程序。

