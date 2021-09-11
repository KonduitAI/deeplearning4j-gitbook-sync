---
description: An overview of the core deeplearning4j workflow
---

# The core workflow

## Introduction

An end to end workflow involves the following:

1. Preparing your data
2. Normalization
3. Building a model
4. Tuning a model
5. Preparing for deployment

This page will try to cover considerations for each workflow and link to additional resources for how to handle each step that maybe specific to particular people.

## Preparing your data

Data always needs to be preprocessed.  This means converting data from a raw source of different data types to ndarrays to be processed by a neural network. In the deeplearning4j suite there can be a few ways to do this:

1. The datavec module: Using a record reader abstraction, data can be read in batches via a data set iterator to train models
2. Pre process using embedded python code in python4j: using the python ecosystem such as pandas and python opencv, you can embed python scripts and output numpy arrays for training
3. Custom java code: using 3rd party libraries such as [tablesaw](https://jtablesaw.github.io/tablesaw/) and [javacv](https://github.com/bytedeco/javacv)

We recommend the following for the various data types:

1. CSV: The CSV record reader in datavec is fairly good for this if you have a lot of data. The reason is the record readers assume that the data you are using is too large to fit in memory. If you have a smaller dataset that can fit in memory you can look at our [tablesaw example](https://github.com/eclipse/deeplearning4j-examples/blob/master/data-pipeline-examples/src/main/java/org/deeplearning4j/datapipelineexamples/tablesaw/TablesawCSVExample.java#L51). If you have a large amount of CSV data then our example [here ](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/quickstart/modeling/feedforward/classification/IrisClassifier.java#L62)should work well.
2. Images: The native image loader and image record reader based on javacv handles loading images of any format and are easily converted to labeled image datasets. We have a comprehensive image example [here](https://github.com/eclipse/deeplearning4j-examples/blob/master/data-pipeline-examples/src/main/java/org/deeplearning4j/datapipelineexamples/formats/image/ImagePipelineExample.java).
3. NLP:  The DL4J suite has a core tokenizer api where a user can supply a tokenizer and build an iterator from that. A combination of that interface and something like our [BERT iterator](https://github.com/eclipse/deeplearning4j/blob/master/deeplearning4j/deeplearning4j-nlp-parent/deeplearning4j-nlp/src/main/java/org/deeplearning4j/iterator/BertIterator.java) allow usage of the latest transformer models. If you are looking for word2vec, then we also have examples for that as well [here](https://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/advanced/modelling/textclassification).
4. Audio:   We do have a midi example [here](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/wip/advanced/modelling/melodl4j/MidiMelodyExtractor.java). Audio should be treated as time series. For your workflow, javacpp \(which our ndarray library nd4j supports internally\) has [ffmpeg bindings](https://github.com/bytedeco/javacpp-presets/tree/master/ffmpeg). Due to licensing restrictions for the project \(basically no gpl code\) we can not directly include ffmpeg in the project, but you are welcome to ask questions on the community forums.
5. Video: Dl4j does not directly support video, but does have 3d convolutional layers for processing video frames. It is suggested to use javacv or ffmpeg mentioned above to process video and convert them in to frames. Please use our [forums](https://community.konduit.ai/) for additional support.

Once you have figured out how you will convert your data, you will need to figure out how to split it up in to training and validation sets. Dl4j allows you to do this in a few ways.

If all of your data is in memory, you can use our dataset api's split test and train api.

An example of that workflow maybe found [here](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/dl4j-examples/src/main/java/org/deeplearning4j/examples/advanced/features/metadata/CSVExampleEvaluationMetaData.java#L74). If your data may not fit in memory, it maybe worth looking in to our minibatch pipelines and ways of creating your test train splits over minibatches. Our image examples cover [this ](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/dl4j-examples/src/main/java/org/deeplearning4j/examples/advanced/modelling/densenet/DenseNetMain.java#L88). For larger input data like images, it is highly suggested to do minibatch partitioning of your data.



## Normalization

Once your input data has been created and converted to ndarrays, you still need to decide how to normalize your data. DL4J has a set of normalizers that cover the standard preprocessing, this includes:

1. [Zero mean unit variance](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/dl4j-examples/src/main/java/org/deeplearning4j/examples/wip/quickstart/modelling/AnimalClassifier.java#L108)
2. [Scale zero to 1](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/dl4j-examples/src/main/java/org/deeplearning4j/examples/quickstart/modeling/feedforward/regression/CSVDataModel.java#L78) - note that this can also be used to scale to a min and max like for images in this case being between 1 and 255.

Normalizers, like models upcoming can be saved and loaded as part of your pipeline. Models must have their accompanying normalizers even during deployment. An example of serializing normalizers can be found [here](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/dl4j-examples/src/main/java/org/deeplearning4j/examples/advanced/modelling/sequenceanomalydetection/SequenceAnomalyDetection.java#L82).



## Building a model

Once you have figured out how you will serialize your data as ndarrays you need to figure out how you will want to build your model.

When building a model, you can choose one of the following:

1. Train a model using the higher level dl4j interface. One quick example can be found [here](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/mvn-project-template/src/main/java/org/deeplearning4j/examples/sample/LeNetMNIST.java#L69).
2. Train a model using samediff: lower level but more flexible. An example can be found [here](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/samediff-examples/src/main/java/org/nd4j/examples/samediff/quickstart/modeling/MNISTFeedforward.java#L52).
3. Import a model from another framework such as tensorflow,keras or pytorch.

If you are going to import a model, there are a few things to be aware of. 

1. Tensorflow import: This uses samediff. Samediff has 2 forms of tensorflow import. The new version is the recommended path forward which uses a more extensible model import framework.
2. Pytorch: Right now, it is required to import pytorch models via onnx. Please use pytorch's onnx model export to import a pytorch model in to deeplearning4j
3. Keras: The keras h5 format integration is a bit older and uses the higher level dl4j interface. Keras model import for non sequential models use the computation graph. An example can be found [here](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/tensorflow-keras-import-examples/src/main/java/org/deeplearning4j/modelimportexamples/keras/advanced/deepmoji/ImportDeepMoji.java#L66). Sequential models can be found [here](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/tensorflow-keras-import-examples/src/main/java/org/deeplearning4j/modelimportexamples/keras/quickstart/SimpleSequentialMlpImport.java#L78).

For more advanced models, it is suggested that the user pick the samediff framework. Going forward, that will be the preferred way to train and run models. 

When saving a model, make sure you save it. Note that the higher level dl4j interface and samediff also have different file formats. When saving models, note that normalizers above are saved separately. It is advised to save both separately.



## Tuning a model

Tuning a model can be difficult. Our [tuning guide](../../deeplearning4j/how-to-guides/tuning-and-training/) can help navigate this. It uses the deeplearning4j ui to monitor the gradients and ensure that they converge quickly. It is recommended to run the dl4j ui in a separate process to avoid dependency clashes. An example of how to run the UI server in a separate process can be found [here](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/quickstart/features/userinterface/RemoteUIExample.java).

When evaluating models, it is suggested to pair the workflow here with the data set splitting considerations above. Our evaluation API takes in ndarrays and tracks evaluations in bits. An example of the higher level dl4j interface's evaluate call can be shown [here](https://github.com/eclipse/deeplearning4j-examples/blob/165f406763330d5e7f8ce842e76d4376e24ff0d1/dl4j-examples/src/main/java/org/deeplearning4j/examples/quickstart/modeling/convolution/LeNetMNISTReLu.java#L171).

A samediff model also has a similar evaluate call. In samediff, you pass in an evaluation object in to a training configuration. Results for the validation set will be streamed in to this object. An example can be found [here](https://github.com/eclipse/deeplearning4j-examples/blob/22197bd797661aac4b9a4871ea4314d43231383e/samediff-examples/src/main/java/org/nd4j/examples/samediff/quickstart/modeling/MNISTCNN.java#L88).



## Deploying a model

When deploying a machine learning model, the first consideration is to figure out what you are deploying. Generally a model deployment contains:

1. A normalizer file which is loaded and used during inference
2. A model file \(either a dl4j zip file or a samediff flatbuffers file\)
3. Data pipeline code that converts raw data from production to an appropriate format \(usually ndarrays\) for consumption by the neural network.

These 3 aspects of a deployment should all be treated as software assets just like code and be versioned. Optionally, a user may want to consider how to implement versioned deployments. There are a number of tools that can handle this.

After a model has been built and deployed, usually the next thing users will want to do are setup the environment in which the model will run. One immediate suggestion is to optimize your dependencies.  
Since the whole deeplearning4j suite heavily relies on javacpp for its underlying dependencies, [this guide](../how-to-guides/developer-docs/javacpp.md#dl-4-j-and-javacpp-overview) is recommended reading as next steps for optimizing your binaries. 

Another consideration is performance. Depending on the nd4j backend you pick and the cpus you are deploying on, you may be able to add specialized performance increases such as:

1. Helpers: Accelerated libraries for faster platform specific math routines including [onednn, armcompute, and cudnn.](../../libnd4j/reference/helpers-overview-cudnn-onednn-armcompute.md)
2. Avx: We pre compile our binaries for specific intel cpus including avx2 and avx512. Various classifiers are available for developers which can be found [here](https://repo1.maven.org/maven2/org/nd4j/nd4j-native/1.0.0-M1.1/).
3. Compatibility: if you need to run on a very old linux, we also provide a centos 6 compatible compat classifier.

For building deployment pipelines, it is recommended to use [konduit-serving](https://github.com/KonduitAI/konduit-serving) which is built on the same technology and is usually co released alongside deeplearning4j. 

If you are going to just be deploying a model embedded in your application, then please remember the above artifacts for a model deployment when including resources for your micro service.



