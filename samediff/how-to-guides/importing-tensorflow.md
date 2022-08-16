# Importing Tensorflow

## What models can be imported into SameDiff

Currently SameDiff supports the import of TensorFlow frozen graphs through the various SameDiff.importFrozenTF methods. TensorFlow documentation on frozen models can be found [here](https://www.tensorflow.org/guide/saved\_model#the\_savedmodel\_format\_on\_disk).

```java
import org.nd4j.autodiff.SameDiff.SameDiff;
SameDiff sd = SameDiff.importFrozenTF(modelFile);
```

## Finding the model input/outputs and running inference

After you import the TensorFlow model there are 2 ways to find the inputs and outputs. The first method is to look at the output of

```java
 sd.summary();
```

Where the input variables are the output of no ops, and the output variables are the input of no ops. Another way to find the inputs is

```java
List<String> inputs = sd.inputs();
```

To run inference use:

```java
INDArray out = sd.batchOutput()
    .input(inputName, inputArray)
    .output(outputs)
    .execSingle();
```

For multiple outputs, use `exec()` instead of `execSingle()`, to return a `Map<String,INDArray>` of outputs instead. Alternatively, you can use methods such as `SameDiff.output(Map<String, INDArray> placeholders, String... outputs)` to get the same output.

## Import Validation

We have a TensorFlow graph analyzing utility which will report any missing operations (operations that still need to be implemented) [here](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-api-parent/nd4j-api/src/main/java/org/nd4j/imports/TensorFlow/TensorFlowImportValidator.java)

## Advanced: Node Skipping and Import Overrides

It is possible to remove nodes from the network. For example TensorFlow 1.x models can have hard coded dropout layers. See the [BERT Graph test](https://github.com/eclipse/deeplearning4j/blob/master/nd4j/nd4j-backends/nd4j-tests/src/test/java/org/nd4j/imports/TFGraphs/BERTGraphTest.java#L114-L150) for an example.

## List of models known to work with SameDiff.

* [PorV-RNN](https://deeplearning4jblob.blob.core.windows.net/testresources/PorV-RNN\_frozenmodel.pb)
* [alexnet](https://deeplearning4jblob.blob.core.windows.net/testresources/alexnet\_frozenmodel.pb)
* [cifar10\_gan\_85](https://deeplearning4jblob.blob.core.windows.net/testresources/cifar10\_gan\_85\_frozenmodel.pb)
* [deeplab\_mobilenetv2\_coco\_voc\_trainval](http://download.tensorflow.org/models/deeplabv3\_mnv2\_pascal\_trainval\_2018\_01\_29.tar.gz)
* [densenet\_2018\_04\_27](https://storage.googleapis.com/download.tensorflow.org/models/tflite/model\_zoo/upload\_20180427/densenet\_2018\_04\_27.tgz)
* [inception\_resnet\_v2\_2018\_04\_27](https://storage.googleapis.com/download.tensorflow.org/models/tflite/model\_zoo/upload\_20180427/inception\_resnet\_v2\_2018\_04\_27.tgz)
* [inception\_v4\_2018\_04\_27](https://storage.googleapis.com/download.tensorflow.org/models/tflite/model\_zoo/upload\_20180427/inception\_v4\_2018\_04\_27.tgz)
* [labels](https://github.com/KonduitAI/dl4j-test-resources/tree/master/src/main/resources/tf\_graphs/zoo\_models/labels)
* [mobilenet\_v1\_0.5\_128](http://download.tensorflow.org/models/mobilenet\_v1\_2018\_02\_22/mobilenet\_v1\_0.5\_128.tgz)
* [mobilenet\_v2\_1.0\_224](http://download.tensorflow.org/models/tflite\_11\_05\_08/mobilenet\_v2\_1.0\_224.tgz)
* [nasnet\_mobile\_2018\_04\_27](https://storage.googleapis.com/download.tensorflow.org/models/tflite/model\_zoo/upload\_20180427/nasnet\_mobile\_2018\_04\_27.tgz)
* [resnetv2\_imagenet\_frozen\_graph](http://download.tensorflow.org/models/official/resnetv2\_imagenet\_frozen\_graph.pb)
* [squeezenet\_2018\_04\_27](https://storage.googleapis.com/download.tensorflow.org/models/tflite/model\_zoo/upload\_20180427/squeezenet\_2018\_04\_27.tgz)
* [temperature\_bidirectional\_63](https://deeplearning4jblob.blob.core.windows.net/testresources/temperature\_bidirectional\_63\_frozenmodel.pb)
* [temperature\_stacked\_63](https://deeplearning4jblob.blob.core.windows.net/testresources/temperature\_stacked\_63\_frozenmodel.pb)
* [text\_gen\_81](https://deeplearning4jblob.blob.core.windows.net/testresources/text\_gen\_81\_frozenmodel.pb)

## Operations Coverage

SameDiff’s TensorFlow import is still being developed, and does not yet have support for every single operation and datatype in TensorFlow. Almost all of the common/standard operations are importable and tested, however - including almost everything in the tf, tf.math, tf.layers, tf.losses, tf.bitwise and tf.nn namespaces. The majority of existing pretrained models out there should be importable into SameDiff.

If you run into an operation that can’t be imported, feel free to [open an issue](https://github.com/eclipse/deeplearning4j/issues).
