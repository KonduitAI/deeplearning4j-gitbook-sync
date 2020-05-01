---
description: 如何使用Eclipse Deeplearning4j在Android上创建IRIS分类器。
---

# Android分类器

## IRIS分类器演示

示例应用程序使用Anderson的Iris数据集在设备上训练一个小的神经网络，用于Iris花类型分类。要更深入地了解如何为DL4J优化android，请参阅此处的先决条件和配置文档。此应用程序有一个简单的用户界面，可以从用户处测量花瓣长度、花瓣宽度、萼片长度和萼片宽度，并返回测量值属于三种类型_Iris_（_Iris serosa_, _Iris versicolor_, 和 _Iris virginica_）之一的概率。一个数据集包括150个测量值（每种_Iris_类型50个）并且训练模型需要5-20秒，视设备而定。

内容

* [设置依赖项](android-fen-lei-qi.md#she-zhi-yi-lai-xiang)
* [在后台线程上建立神经网络](android-fen-lei-qi.md#zai-hou-tai-xian-cheng-shang-jian-li-shen-jing-wang-luo)
* [准备训练数据集和用户输入](android-fen-lei-qi.md#zhun-bei-xun-lian-shu-ju-ji-he-yong-hu-shu-ru)
* [神经网络的建立与训练](android-fen-lei-qi.md#shen-jing-wang-luo-de-jian-li-yu-xun-lian)
* [更新用户界面](android-fen-lei-qi.md#geng-xin-yong-hu-jie-mian)
* [结论](android-fen-lei-qi.md#jie-lun)

## DL4JIrisClassifierDemo

## [设置依赖项](android-fen-lei-qi.md)

Deeplearning4J应用程序需要build.gradle文件中的几个依赖项。Deeplearning库又依赖于ND4J和OpenBLAS库，因此这些库也必须添加到依赖关系声明中。从Android Studio 3.0开始，还需要定义annotationProcessors，需要依赖于-x86或-arm处理器。

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

编译这些依赖项涉及大量文件，因此有必要在defaultConfig中将multiDexEnabled设置为true。

```java
multiDexEnabled true
```

junit模块版本中的冲突通常会导致以下错误：_&gt; Conflict with dependency 'junit:junit' in project ':app'_。应用程序（4.8.2）和测试应用程序（4.12）的解析版本不同。这可以通过强制所有junit模块使用相同的版本来抑制：

```java
configurations.all {
    resolutionStrategy.force 'junit:junit:4.12'
}
```

## [在后台线程上建立神经网络](android-fen-lei-qi.md)

即使是像本例中这样简单的神经网络训练，也需要相当大的处理器功率，这在移动设备上是受限的。因此，必须使用一个后台线程来构建和训练神经网络，然后将输出返回给主线程以更新UI。在本例中，我们将使用AsyncTask，它接受来自UI的输入测量，并将它们作为double类型传递给doInBackground\(\)方法。首先，让我们获取对UI布局中editTexts的引用，该UI布局接受onCreate方法内部的iris测量。然后onClickListener将执行我们的asyncTask，将用户输入的测量传递给它，并显示一个进度条，直到我们在onPostExecute\(\)中再次隐藏它为止。

```java
public class MainActivity extends AppCompatActivity {


@Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //获取对进行测量的editTexts的引用
        final EditText PL = (EditText) findViewById(R.id.editText);
        final EditText PW = (EditText) findViewById(R.id.editText2);
        final EditText SL = (EditText) findViewById(R.id.editText3);
        final EditText SW = (EditText) findViewById(R.id.editText4);

      
        //onclick捕获输入并启动asyncTask
        Button button = (Button) findViewById(R.id.button);

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                final double pl = Double.parseDouble(PL.getText().toString());
                final double pw = Double.parseDouble(PW.getText().toString());
                final double sl = Double.parseDouble(SL.getText().toString());
                final double sw = Double.parseDouble(SW.getText().toString());

                AsyncTaskRunner runner = new AsyncTaskRunner();

                //将测量作为参数传递给AsyncTask
                runner.execute(pl,pw,sl,sw);

                ProgressBar bar = (ProgressBar) findViewById(R.id.progressBar);
                bar.setVisibility(View.VISIBLE);
            }
        });
        }
```

现在让我们编写AsyncTask&lt;_Params_, _Progress_, _Results_&gt;.。AsyncTask需要有Double类型的参数才能从UI接收十进制值测量值。结果类型设置为INDArray，它从doInBackground\(\)方法返回并传递给onPostExecute\(\)方法以更新UI。NDArrays由ND4J库提供，本质上是具有给定维数的n维数组。有关NDArrays的更多信息，请参见[https://nd4j.org/userguide](https://nd4j.org/userguide).

```java
private class AsyncTaskRunner extends AsyncTask<Double, Integer, INDArray> {

    // 在调用后台线程之前在UI中运行
    @Override
    protected void onPreExecute() {
        super.onPreExecute();

        ProgressBar bar = (ProgressBar) findViewById(R.id.progressBar);
        bar.setVisibility(View.INVISIBLE);
    }
```

## [准备训练数据集和用户输入](android-fen-lei-qi.md)

doInBackground\(\)方法将处理训练数据的格式化、神经网络的构造、网络的训练以及通过训练模型对输入数据的分析。用户输入只有4个值，因此我们可以使用putScalar\(\)方法将这些值直接添加到1x4 INDArray中。训练数据要大得多，必须通过迭代for循环从CSV列表转换为矩阵。 

训练数据以两个数组的形式存储在应用程序中，一个数组用于名为_irisData_的Iris测量，其中包含150个Iris测量的列表，另一个数组用于名为labelData的Iris类型标签。它们将分别转换为150x4和150x3矩阵，以便它们可以转换为INDArray对象，神经网络将用于训练。

```java
    //这是我们神经网络的主要后台线程
    @Override
    protected String doInBackground(Double... params) {
    //从params中获取doubles，params是一个数组，因此它们是0,1,2,3
        double pld = params[0];
        double pwd = params[1];
        double sld = params[2];
        double swd = params[3];

        //为用户测量创建输入INDArray
        INDArray actualInput = Nd4j.zeros(1,4);
        actualInput.putScalar(new int[]{0,0}, pld);
        actualInput.putScalar(new int[]{0,1}, pwd);
        actualInput.putScalar(new int[]{0,2}, sld);
        actualInput.putScalar(new int[]{0,3}, swd);

        //将iris数据转换成150x4矩阵
        int row=150;
        int col=4;
        double[][] irisMatrix=new double[row][col];
        int i = 0;
        for(int r=0; r<row; r++){
            for( int c=0; c<col; c++){
        irisMatrix[r][c]=com.example.jmerwin.irisclassifier.DataSet.irisData[i++];
            }
        }

        //现在对标签数据执行同样的操作
        int rowLabel=150;
        int colLabel=3;
        double[][] twodimLabel=new double[rowLabel][colLabel];
        int ii = 0;
        for(int r=0; r<rowLabel; r++){
            for( int c=0; c<colLabel; c++){
                twodimLabel[r][c]=com.example.jmerwin.irisclassifier.DataSet.labelData[ii++];
            }
        }

       //直接将数据矩阵转换为训练INDArrays
        INDArray trainingIn = Nd4j.create(irisMatrix);
        INDArray trainingOut = Nd4j.create(twodimLabel);
```

## [神经网络的建立与训练](android-fen-lei-qi.md)

现在我们的数据已经准备好了，我们可以用一个隐藏层来构建一个简单的多层感知器。DenseLayer类用于创建网络的输入层和隐藏层，OutputLayer类用于输出层。输入INDArray中的列数必须等于输入层（nIn）中的神经元数。隐藏层输入中的神经元数必须等于输入层的输出数组（nOut）数。最后，outputLayer输入应该与hiddenLayer输出匹配。输出必须等于可能的分类数，即3。

```java
    //定义网络层
    DenseLayer inputLayer = new DenseLayer.Builder()
            .nIn(4)
            .nOut(3)
            .name("Input")
            .build();

    DenseLayer hiddenLayer = new DenseLayer.Builder()
            .nIn(3)
            .nOut(3)
            .name("Hidden")
            .build();

    OutputLayer outputLayer = new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
            .nIn(3)
            .nOut(3)
            .name("Output")
            .activation(Activation.SOFTMAX)
            .build();
```

下一步是使用nccBuilder构建神经网络。下面选择的训练参数是标准的。要了解有关优化网络训练的更多信息，请参阅deeplearning4j.org。

```java
    NeuralNetConfiguration.Builder nncBuilder = new NeuralNetConfiguration.Builder();
    long seed = 6;
    nncBuilder.seed(seed);
    nncBuilder.activation(Activation.TANH);
    nncBuilder.weightInit(WeightInit.XAVIER);

    NeuralNetConfiguration.ListBuilder listBuilder = nncBuilder.list();
    listBuilder.layer(0, inputLayer);
    listBuilder.layer(1, hiddenLayer);
    listBuilder.layer(2, outputLayer);

    listBuilder.backprop(true);

    MultiLayerNetwork myNetwork = new MultiLayerNetwork(listBuilder.build());
    myNetwork.init();

    //从INDArrays创建数据集并训练网络
    DataSet myData = new DataSet(trainingIn, trainingOut);
    for(int l=0; l<=1000; l++) {
    myNetwork.fit(myData);
    }

    //根据模型评估输入数据
    INDArray actualOutput = myNetwork.output(actualInput);
    Log.d("myNetwork Output ", actualOutput.toString());

    //在这里，我们将INDArray返回到onPostExecute
    //用于更新用户界面
    return actualOutput;
}
```

## [更新用户界面](android-fen-lei-qi.md)

一旦神经网络的训练和用户测量的分类完成，doInBackground\(\)方法将完成，onPostExecute\(\)将访问主线程和UI，允许我们用分类结果更新UI。注意，概率报告的小数位数可以通过设置DecimalFormat模式来控制。

```java
   //这是我们用分类结果更新UI的地方
    @Override
    protected void onPostExecute(INDArray result) {
        super.onPostExecute(result);

    //完成后隐藏进度条
    ProgressBar bar = (ProgressBar) findViewById(R.id.progressBar);
    bar.setVisibility(View.INVISIBLE);

    //获取三个概率
    Double first = result.getDouble(0,0);
    Double second = result.getDouble(0,1);
    Double third = result.getDouble(0,2);

    //用输出更新UI
    TextView setosa = (TextView) findViewById(R.id.textView11);
    TextView versicolor = (TextView) findViewById(R.id.textView12);
    TextView virginica = (TextView) findViewById(R.id.textView13);

    //使用DecimalFormat将double限制为两个小数
    DecimalFormat df2 = new DecimalFormat(".##");

    //在UI中设置textViews的文本以显示概率
    setosa.setText(String.valueOf(df2.format(first)));
    versicolor.setText(String.valueOf(df2.format(second)));
    virginica.setText(String.valueOf(df2.format(third)));

    }
```

## [结论](android-fen-lei-qi.md)

希望本教程说明了DL4J与Android的兼容性如何使它在移动设备上构建、训练和评估神经网络变得容易。我们使用一个简单的UI从测量中获取输入值，然后在AsyncTask中将它们作为参数传递。处理器密集的数据准备、网络层构建、模型训练和用户数据评估步骤都是在后台线程的doInBackground\(\)方法中执行的，保持了设备的稳定和响应。完成后，我们将输出INDArray作为AsyncTask Results传递给onPostExecute\(\)，更新UI以演示分类结果。移动设备处理能力和电池寿命的限制使得训练健壮的多层网络变得有些不可行。为了解决这个限制，我们接下来将看一个Android应用程序示例，该应用程序在初始模型训练之后将经过训练的模型保存在设备上以获得更快的性能。

 [此处](https://github.com/eclipse/deeplearning4j-examples/tree/master/android/DL4JIrisClassifierDemo)提供了此示例的完整代码。

