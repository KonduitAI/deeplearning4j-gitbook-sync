---
title: Deeplearning4j Model Persistence
short_title: Model Persistence
description: Saving and loading of neural networks.
category: Models
weight: 10
---

# Model Persistence

MultiLayerNetwork and ComputationGraph both have save and load methods.

You can save/load a MultiLayerNetwork using:

```java
MultiLayerNetwork net = ...
net.save(new File("...");

MultiLayerNetwork net2 = MultiLayerNetwork.load(new File("..."), true);
```

Similarly, you can save/load a ComputationGraph using:

```java
ComputationGraph net = ...
net.save(new File("..."));

ComputationGraph net2 = ComputationGraph.load(new File("..."), true);
```

Internally, these methods use the `ModelSerializer` class, which handles loading and saving models. There are two methods for saving models shown in the examples through the link. The first example saves a normal multilayer network, the second one saves a [computation graph](https://deeplearning4j.org/docs/latest/deeplearning4j-nn-computationgraph).

Here is a [basic example](https://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/modelsaving) with code to save a computation graph using the `ModelSerializer` class, as well as an example of using ModelSerializer to save a neural net built using MultiLayer configuration.

## RNG Seed

If your model uses probabilities \(i.e. DropOut/DropConnect\), it may make sense to save it separately, and apply it after model is restored; i.e:

```java
 Nd4j.getRandom().setSeed(12345);
 ModelSerializer.restoreMultiLayerNetwork(modelFile);
```

This will guarantee equal results between sessions/JVMs.

## ModelSerializer

[\[source\]](https://github.com/eclipse/deeplearning4j/tree/master/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/util/ModelSerializer.java)

Utility class suited to save/restore neural net models

**writeModel**

```text
public static void writeModel(@NonNull Model model, @NonNull File file, boolean saveUpdater) throws IOException
```

Write a model to a file

* param model the model to write
* param file the file to write to
* param saveUpdater whether to save the updater or not
* throws IOException

**writeModel**

```text
public static void writeModel(@NonNull Model model, @NonNull File file, boolean saveUpdater,DataNormalization dataNormalization) throws IOException
```

Write a model to a file

* param model the model to write
* param file the file to write to
* param saveUpdater whether to save the updater or not
* param dataNormalization the normalizer to save \(optional\)
* throws IOException

**writeModel**

```text
public static void writeModel(@NonNull Model model, @NonNull String path, boolean saveUpdater) throws IOException
```

Write a model to a file path

* param model the model to write
* param path the path to write to
* param saveUpdater whether to save the updater or not
* throws IOException

**writeModel**

```text
public static void writeModel(@NonNull Model model, @NonNull OutputStream stream, boolean saveUpdater)
            throws IOException
```

Write a model to an output stream

* param model the model to save
* param stream the output stream to write to
* param saveUpdater whether to save the updater for the model or not
* throws IOException

**writeModel**

```text
public static void writeModel(@NonNull Model model, @NonNull OutputStream stream, boolean saveUpdater,DataNormalization dataNormalization)
            throws IOException
```

Write a model to an output stream

* param model the model to save
* param stream the output stream to write to
* param saveUpdater whether to save the updater for the model or not
* param dataNormalization the normalizer ot save \(may be null\)
* throws IOException

**restoreMultiLayerNetwork**

```text
public static MultiLayerNetwork restoreMultiLayerNetwork(@NonNull File file) throws IOException
```

Load a multi layer network from a file

* param file the file to load from
* return the loaded multi layer network
* throws IOException

**restoreMultiLayerNetwork**

```text
public static MultiLayerNetwork restoreMultiLayerNetwork(@NonNull File file, boolean loadUpdater)
            throws IOException
```

Load a multi layer network from a file

* param file the file to load from
* return the loaded multi layer network
* throws IOException

**restoreMultiLayerNetwork**

```text
public static MultiLayerNetwork restoreMultiLayerNetwork(@NonNull InputStream is, boolean loadUpdater)
            throws IOException
```

Load a MultiLayerNetwork from InputStream from an input stream  
Note: the input stream is read fully and closed by this method. Consequently, the input stream cannot be re-used.

* param is the inputstream to load from
* return the loaded multi layer network
* throws IOException
* see \#restoreMultiLayerNetworkAndNormalizer\(InputStream, boolean\)

**restoreMultiLayerNetwork**

```text
public static MultiLayerNetwork restoreMultiLayerNetwork(@NonNull InputStream is) throws IOException
```

Restore a multi layer network from an input stream  
Note: the input stream is read fully and closed by this method. Consequently, the input stream cannot be re-used.

* param is the input stream to restore from
* return the loaded multi layer network
* throws IOException
* see \#restoreMultiLayerNetworkAndNormalizer\(InputStream, boolean\)

**restoreMultiLayerNetwork**

```text
public static MultiLayerNetwork restoreMultiLayerNetwork(@NonNull String path) throws IOException
```

Load a MultilayerNetwork model from a file

* param path path to the model file, to get the computation graph from
* return the loaded computation graph
* throws IOException

**restoreMultiLayerNetwork**

```text
public static MultiLayerNetwork restoreMultiLayerNetwork(@NonNull String path, boolean loadUpdater)
            throws IOException
```

Load a MultilayerNetwork model from a file

* param path path to the model file, to get the computation graph from
* return the loaded computation graph
* throws IOException

**restoreComputationGraph**

```text
public static ComputationGraph restoreComputationGraph(@NonNull String path) throws IOException
```

Restore a MultiLayerNetwork and Normalizer \(if present - null if not\) from the InputStream. Note: the input stream is read fully and closed by this method. Consequently, the input stream cannot be re-used.

* param is Input stream to read from
* param loadUpdater Whether to load the updater from the model or not
* return Model and normalizer, if present
* throws IOException If an error occurs when reading from the stream

**restoreComputationGraph**

```text
public static ComputationGraph restoreComputationGraph(@NonNull String path, boolean loadUpdater)
            throws IOException
```

Load a computation graph from a file

* param path path to the model file, to get the computation graph from
* return the loaded computation graph
* throws IOException

**restoreComputationGraph**

```text
public static ComputationGraph restoreComputationGraph(@NonNull InputStream is, boolean loadUpdater)
            throws IOException
```

Load a computation graph from a InputStream

* param is the inputstream to get the computation graph from
* return the loaded computation graph
* throws IOException

**restoreComputationGraph**

```text
public static ComputationGraph restoreComputationGraph(@NonNull InputStream is) throws IOException
```

Load a computation graph from a InputStream

* param is the inputstream to get the computation graph from
* return the loaded computation graph
* throws IOException

**restoreComputationGraph**

```text
public static ComputationGraph restoreComputationGraph(@NonNull File file) throws IOException
```

Load a computation graph from a file

* param file the file to get the computation graph from
* return the loaded computation graph
* throws IOException

**restoreComputationGraph**

```text
public static ComputationGraph restoreComputationGraph(@NonNull File file, boolean loadUpdater) throws IOException
```

Restore a ComputationGraph and Normalizer \(if present - null if not\) from the InputStream. Note: the input stream is read fully and closed by this method. Consequently, the input stream cannot be re-used.

* param is Input stream to read from
* param loadUpdater Whether to load the updater from the model or not
* return Model and normalizer, if present
* throws IOException If an error occurs when reading from the stream

**taskByModel**

```text
public static Task taskByModel(Model model)
```

* param model
* return

**addNormalizerToModel**

```text
public static void addNormalizerToModel(File f, Normalizer<?> normalizer)
```

This method appends normalizer to a given persisted model.

PLEASE NOTE: File should be model file saved earlier with ModelSerializer

* param f
* param normalizer

**addObjectToFile**

```text
public static void addObjectToFile(@NonNull File f, @NonNull String key, @NonNull Object o)
```

Add an object to the \(already existing\) model file using Java Object Serialization. Objects can be restored using {- link \#getObjectFromFile\(File, String\)}

* param f File to add the object to
* param key Key to store the object under
* param o Object to store using Java object serialization

