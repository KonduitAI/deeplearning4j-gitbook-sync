---
description: 如何使用Eclipse Deeplearning4j创建Android图像分类应用程序。
---

# Android图片分类器

## 在Android应用程序中使用Deeplearning4J

内容

* [设置依赖项](android-tu-pian-fen-lei-qi.md#she-zhi-yi-lai-xiang)
* [在Android项目资源中训练和加载MNIST模型](android-tu-pian-fen-lei-qi.md#zai-android-xiang-mu-zi-yuan-zhong-xun-lian-he-jia-zai-mnist-mo-xing)
* [使用异步任务访问训练模型](android-tu-pian-fen-lei-qi.md#shi-yong-yi-bu-ren-wu-fang-wen-xun-lian-mo-xing)
* [从用户输入处理图像](android-tu-pian-fen-lei-qi.md#cong-yong-hu-shu-ru-chu-li-tu-xiang)
* [更新用户界面](android-tu-pian-fen-lei-qi.md#geng-xin-yong-hu-jie-mian)
* [结论](android-tu-pian-fen-lei-qi.md#jie-lun)

## DL4JImageRecognitionDemo

此示例应用程序使用在28x28灰度0..255像素值的手写数字0..9的标准MNIST数据集上训练的神经网络。应用程序用户界面允许用户在设备屏幕上绘制一个数字，然后根据经过训练的网络进行测试。输出显示最可能的数值和概率分数。本教程将介绍如何在Android应用程序中使用经过训练的神经网络，如何处理用户生成的图像，以及如何将结果从后台线程输出到UI。关于构建DL4J Android应用程序的一般先决条件的更多信息可以在这里找到。

## [设置依赖项](android-tu-pian-fen-lei-qi.md)

Deeplearning4J应用程序需要build.gradle文件中特定于应用程序的依赖项。Deeplearning库又依赖于ND4J和OpenBLAS库，因此这些库也必须添加到依赖关系声明中。从Android Studio 3.0开始，还需要定义annotationProcessors，因此，如果您在Android studio3.0或更高版本中工作，则应根据您的设备包括-x86或-arm处理器的依赖项。请注意，这两个应用程序都可以包含而不发生冲突，就像在示例应用程序中所做的那样。

```groovy
implementation (group: 'org.deeplearning4j', name: 'deeplearning4j-core', version: '{{page.version}}') {
    exclude group: 'org.bytedeco', module: 'opencv-platform'
    exclude group: 'org.bytedeco', module: 'leptonica-platform'
    exclude group: 'org.bytedeco', module: 'hdf5-platform'
    exclude group: 'org.nd4j', module: 'nd4j-base64'
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

implementation 'com.google.code.gson:gson:2.8.2'
annotationProcessor 'org.projectlombok:lombok:1.16.16'

//This corrects for a junit version conflict.
configurations.all {
    resolutionStrategy.force 'junit:junit:4.12'
}
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

## [在Android项目资源中训练和加载Mnist模型](android-tu-pian-fen-lei-qi.md)

使用神经网络需要相当大的处理器功率，这在移动设备上是受限的。因此，必须使用一个后台线程加载训练好的神经网络，并使用AsyncTask测试用户绘制的图像。在这个应用程序中，我们将在主线程上运行canvas.draw代码，并使用AsyncTask从内存中加载绘制的图像，并在后台线程上用经过训练的模型对其进行测试。首先，让我们看看如何保存我们将在应用程序中使用的经过训练的神经网络。

首先，您需要遵循DeepLearning4j快速入门指南，在台式计算机上建立、训练和保存神经网络模型。训练和保存此应用程序中使用的MNIST模型的DL4J示例是MnistImagePipelineExampleSave.java，它包含在上述快速入门指南中。MNIST演示的代码也可以在[这里](https://gist.github.com/tomthetrainer/7cb2fbc14a5c631a567a98c3134f7dd6)找到。运行此演示将训练MNIST神经网络模型，并将其保存为dl4j-examples目录的dl4j\target文件夹中的“_trained\_mnist\_model.zip"_”。然后，可以复制该文件并将其保存在Android项目的raw文件夹中。

## [使用异步任务访问训练模型](android-tu-pian-fen-lei-qi.md)

现在让我们从编写AsyncTask&lt;_Params_, _Progress_, _Results_&gt;开始，在后台线程上加载并使用神经网络。异步任务将使用参数类型。_Params_类型设置为String，它将在执行保存的图像时将其路径传递给asyncTask。此路径将在doInBackground\(\)方法中用于定位和加载经过训练的MNIST模型。Results参数是INDArray类型，它将存储来自神经网络的结果，并将其传递给onPostExecute方法，该方法可以访问用于更新UI的主线程。有关INDArray的更多信息，请参见https://nd4j.org/userguide。注意，AsyncTask要求我们重写另外两个方法（onProgressUpdate和onPostExecute方法），稍后我们将在演示中讨论这些方法。

```java
private class AsyncTaskRunner extends AsyncTask<String, Integer, INDArray> {

       //在调用后台线程之前在UI中运行。
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
        }

        @Override
        protected INDArray doInBackground(String... params) {
            //主后台线程，这将加载模型并测试输入图像
            //图像的尺寸设置在这里
            int height = 28;
            int width = 28;
            int channels = 1;

            //现在我们使用try/catch块从raw文件夹加载模型
            try {
                // Load the pretrained network.
                InputStream inputStream = getResources().openRawResource(R.raw.trained_mnist_model);
                MultiLayerNetwork model = ModelSerializer.restoreMultiLayerNetwork(inputStream);

                //加载要测试的图像文件
                File f=new File(absolutePath, "drawn_image.jpg");

               //使用nativeImageLoader转换为数值矩阵
                NativeImageLoader loader = new NativeImageLoader(height, width, channels);

                //将图像放入INDArray
                INDArray image = loader.asMatrix(f);

                //值需要缩放
                DataNormalization scalar = new ImagePreProcessingScaler(0, 1);

                //然后调用图像数据集上的标量
                scalar.transform(image);

           
                //通过神经网络存储在输出数组中
                output = model.output(image);

            } catch (IOException e) {
                e.printStackTrace();
            }
            return output;
        }
```

## [从用户输入处理图像](android-tu-pian-fen-lei-qi.md)

现在，让我们为将在主线程上运行的绘图画布添加代码，并允许用户在屏幕上绘制一个数字。这是一个通用绘图程序，作为MainActivity中的内部类编写。它扩展了View并重写了一系列方法。图形保存到内部内存中，并在case MotionEvent.ACTION上的onTouchEvent case语句中使用传递给它的图像路径执行AsyncTask。这有一个流线型操作，即在用户完成绘图后自动返回图像的结果。

```java
    //图形输入代码
    public class DrawingView extends View {

        private Path    mPath;
        private Paint   mBitmapPaint;
        private Paint   mPaint;
        private Bitmap  mBitmap;
        private Canvas  mCanvas;

        public DrawingView(Context c) {
            super(c);

            mPath = new Path();
            mBitmapPaint = new Paint(Paint.DITHER_FLAG);
            mPaint = new Paint();
            mPaint.setAntiAlias(true);
            mPaint.setStrokeJoin(Paint.Join.ROUND);
            mPaint.setStrokeCap(Paint.Cap.ROUND);
            mPaint.setStrokeWidth(60);
            mPaint.setDither(true);
            mPaint.setColor(Color.WHITE);
            mPaint.setStyle(Paint.Style.STROKE);
        }

        @Override
        protected void onSizeChanged(int W, int H, int oldW, int oldH) {
            super.onSizeChanged(W, H, oldW, oldH);
            mBitmap = Bitmap.createBitmap(W, H, Bitmap.Config.ARGB_4444);
            mCanvas = new Canvas(mBitmap);
        }

        @Override
        protected void onDraw(Canvas canvas) {
            canvas.drawBitmap(mBitmap, 0, 0, mBitmapPaint);
            canvas.drawPath(mPath, mPaint);
        }

        private float mX, mY;
        private static final float TOUCH_TOLERANCE = 4;

        private void touch_start(float x, float y) {
            mPath.reset();
            mPath.moveTo(x, y);
            mX = x;
            mY = y;
        }
        private void touch_move(float x, float y) {
            float dx = Math.abs(x - mX);
            float dy = Math.abs(y - mY);
            if (dx >= TOUCH_TOLERANCE || dy >= TOUCH_TOLERANCE) {
                mPath.quadTo(mX, mY, (x + mX)/2, (y + mY)/2);
                mX = x;
                mY = y;
            }
        }
        private void touch_up() {
            mPath.lineTo(mX, mY);
            mCanvas.drawPath(mPath, mPaint);
            mPath.reset();
        }

        @Override
        public boolean onTouchEvent(MotionEvent event) {
            float x = event.getX();
            float y = event.getY();

            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    invalidate();
                    clear();
                    touch_start(x, y);
                    invalidate();
                    break;
                case MotionEvent.ACTION_MOVE:
                    touch_move(x, y);
                    invalidate();
                    break;
                case MotionEvent.ACTION_UP:
                    touch_up();
                    absolutePath = saveDrawing();
                    invalidate();
                    clear();
                    loadImageFromStorage(absolutePath);
                    onProgressBar();
                    //保存图像后立即启动asyncTask
                    AsyncTaskRunner runner = new AsyncTaskRunner();
                    runner.execute(absolutePath);
                    break;

            }
            return true;
        }

        public void clear(){
            mBitmap.eraseColor(Color.TRANSPARENT);
            invalidate();
            System.gc();
        }

    }
```

现在我们需要构建一系列的帮助方法。首先我们将编写saveDrawing\(\)方法。它使用getDrawingCache\(\)从drawingView中获取图形并将其存储为位图。然后我们为位图创建一个名为“drawind\_image.jpg”的文件目录和文件。最后，在try/catch块中使用FileOutputStream将位图写入文件位置。方法返回loadImageFromStorage\(\)方法将使用的文件位置的绝对路径。

```java
public String saveDrawing(){
        drawingView.setDrawingCacheEnabled(true);
        Bitmap b = drawingView.getDrawingCache();

        ContextWrapper cw = new ContextWrapper(getApplicationContext());
        //设置存储路径
        File directory = cw.getDir("imageDir", Context.MODE_PRIVATE);
        //创建imageDir并将文件存储在那里。每个新图形都将覆盖上一个
        File mypath=new File(directory,"drawn_image.jpg");

       //使用fileOutputStream将文件写入try/catch块中的位置
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream(mypath);
            b.compress(Bitmap.CompressFormat.JPEG, 100, fos);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return directory.getAbsolutePath();
    }
```

接下来我们将编写loadImageFromStorage方法，该方法将使用saveDrawing\(\)返回的绝对路径来加载保存的图像，并将其作为输出显示的一部分显示在UI中。它使用try/catch块和FileInputStream在UI布局中将图像设置为ImageView img。

```java
    private void loadImageFromStorage(String path)
    {

        //使用fileOutputStream将文件写入try/catch块中的位置
        try {
            File f=new File(path, "drawn_image.jpg");
            Bitmap b = BitmapFactory.decodeStream(new FileInputStream(f));
            ImageView img=(ImageView)findViewById(R.id.outputView);
            img.setImageBitmap(b);
        }
        catch (FileNotFoundException e)
        {
            e.printStackTrace();
        }

    }
```

我们还需要编写两种方法，从神经网络输出和置信度得分中提取预测数，我们稍后在完成AsyncTask时将调用这两种方法。

```java
    //helper类返回输出数组中的最大值
    public static double arrayMaximum(double[] arr) {
        double max = Double.NEGATIVE_INFINITY;
        for(double cur: arr)
            max = Math.max(max, cur);
        return max;
    }

     //helper类查找最大置信分数的索引（因此是数值）
    public int getIndexOfLargestValue( double[] array )
    {
        if ( array == null || array.length == 0 ) return -1;
        int largest = 0;
        for ( int i = 1; i < array.length; i++ )
        {if ( array[i] > array[largest] ) largest = i;            }
        return largest;
    }
```

最后，我们需要调用一些方法来控制后台线程运行时“进行中…”消息的可见性。当AsyncTask执行时和后台线程完成时在onPostExecute方法中调用这些函数。

```java
    public void onProgressBar(){
        TextView bar = findViewById(R.id.processing);
        bar.setVisibility(View.VISIBLE);
    }

    public void offProgressBar(){
        TextView bar = findViewById(R.id.processing);
        bar.setVisibility(View.INVISIBLE);
    }
```

现在让我们转到onCreate方法来初始化绘图画布并设置一些全局变量。

```java
public class MainActivity extends AppCompatActivity {

    MainActivity.DrawingView drawingView;
    String absolutePath;
    public static INDArray output;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        RelativeLayout parent = findViewById(R.id.layout2);
        drawingView = new MainActivity.DrawingView(this);
        parent.addView(drawingView);
    }
```

## [更新用户界面](android-tu-pian-fen-lei-qi.md)

现在，我们可以通过重写onProgress和onPostExecute方法来完成AsyncTask。AsyncTask的doInBackground方法完成后，分类结果将传递给onPostExecute，onPostExecute有权访问主线程和UI，允许我们用结果更新UI。因为我们不会使用onProgress方法，所以调用它的超类就足够了。

```java
        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);
        }
```

onPostExecute方法将接收一个INDArray，该INDArray将神经网络结果作为概率值的1x10数组，即输入图形是每个可能的数字（0..9）。由此我们需要确定数组的哪一行包含最大的值以及该值的大小。这两个值将决定神经网络将绘图分类为哪个数字以及网络分数的置信度。这些值在UI中将分别称为预测值和置信度。在下面的代码中，使用结果INDArray上的getDouble\(\)方法将INDArray的每个位置的单个值传递给double类型的数组。然后，我们获得对TextViews的引用，这些TextViews将在UI中更新，并调用数组中的帮助方法来返回数组最大值（置信度）和最大值（预测）的索引。注意，我们还需要通过设置DecimalFormat模式来限制概率报告的小数位数。

```java
        @Override
        protected void onPostExecute(INDArray result) {
            super.onPostExecute(result);

            //用于控制输出概率的小数位数
            DecimalFormat df2 = new DecimalFormat(".##");

            //将神经网络输出传输到数组
            double[] results = {result.getDouble(0,0),result.getDouble(0,1),result.getDouble(0,2),
                    result.getDouble(0,3),result.getDouble(0,4),result.getDouble(0,5),result.getDouble(0,6),
                    result.getDouble(0,7),result.getDouble(0,8),result.getDouble(0,9),};

            //找到UI tvs以显示预测值和置信值
            TextView out1 = findViewById(R.id.prediction);
            TextView out2 = findViewById(R.id.confidence);

            //使用下面定义的助手函数显示值
            out2.setText(String.valueOf(df2.format(arrayMaximum(results))));
            out1.setText(String.valueOf(getIndexOfLargestValue(results)));

            //关闭进度测试的助手函数
            offProgressBar();
        }
```

## [结论](android-tu-pian-fen-lei-qi.md)

本教程提供了使用DL4J神经网络在Android应用程序中进行图像识别的基本框架。它说明了如何从原始资源文件加载预先训练的DL4J模型，以及如何测试用户创建与模型相对应的图像。然后，AsyncTask将输出返回到主线程并更新UI。 

[此处](https://github.com/eclipse/deeplearning4j-examples/tree/master/android/DL4JImageRecognitionDemo)提供了此示例的完整代码。

