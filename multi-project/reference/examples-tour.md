---
title: "Examples Tour"
description: "Tour of the dl4j-examples repository — classification, regression, CNN, RNN, and advanced examples"
---

## Tour of the DL4J Examples Repository

The [dl4j-examples](https://github.com/eclipse/deeplearning4j-examples) repository contains runnable, self-contained Java programs covering most of DL4J's functionality. Each example is designed to be readable and educational: the code is annotated, data loading is handled automatically, and results are printed to the console or displayed visually.

This page walks through the major areas of the repository, describes what each example demonstrates, and links to the source code.

---

## Getting the Examples

### Clone and Build

```bash
git clone https://github.com/eclipse/deeplearning4j-examples.git
cd deeplearning4j-examples
mvn clean package -DskipTests
```

### Open in IntelliJ

1. File → Open → select the `deeplearning4j-examples` directory.
2. IntelliJ will detect the Maven project and import it automatically.
3. Wait for indexing to complete, then right-click any example class and choose **Run**.

See the [Quickstart](quickstart.md) for detailed IDE setup instructions.

### Repository Structure

```
deeplearning4j-examples/
├── dl4j-examples/          # Core DL4J examples (most of what you want)
├── datavec-examples/       # Data ingestion and transformation examples
├── dl4j-spark-examples/    # Distributed training on Spark
├── nd4j-examples/          # Raw NDArray and linear algebra examples
└── rl4j-examples/          # Reinforcement learning examples
```

---

## DataVec Examples

DataVec is DL4J's data pipeline library. It handles ingesting raw files (CSV, images, video, audio), applying transformations, and producing `DataSet` objects ready for training. If your data is not in a standard format, DataVec is where you spend the most time.

### IrisAnalysis.java

Loads the canonical Iris flower dataset into a Spark RDD and runs a schema analysis. Good first example of the DataVec analysis API.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/datavec-examples/src/main/java/org/datavec/transform/analysis/IrisAnalysis.java)

### BasicDataVecExample.java

Demonstrates the transform process: loading data into a Spark RDD, defining a `TransformProcess` to filter rows, apply time transformations, and remove columns. The core pattern you will use for any ETL work.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/datavec-examples/src/main/java/org/datavec/transform/basic/BasicDataVecExample.java)

### PrintSchemasAtEachStep.java

Shows how to print the schema at each step of a transform pipeline, which is essential for debugging data issues. When a transform is producing unexpected output, this is the first tool to reach for.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/datavec-examples/src/main/java/org/datavec/transform/debugging/PrintSchemasAtEachStep.java)

### JoinExample.java

Demonstrates joining two separate datasets in DataVec before passing the combined result to a network. Useful when your features come from multiple tables or files.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/datavec-examples/src/main/java/org/datavec/transform/join/JoinExample.java)

### MnistImagePipelineExample.java

Uses `ParentPathLabelGenerator` and `ImagePreProcessingScaler` to load MNIST images from disk and normalize them. Shows the standard pattern for any image loading task.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/dataExamples/MnistImagePipelineExample.java)

### CSVExampleEvaluationMetaData.java

Demonstrates `RecordMetaData`, which tracks the origin of each example. When a network produces errors on specific inputs, this lets you trace back to the original source records.

[Source](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/dataexamples/CSVExampleEvaluationMetaData.java)

---

## Classification Examples

### MLPMnistSingleLayerExample.java

A single hidden-layer feedforward network for MNIST digit classification. The simplest possible end-to-end DL4J program: data loading, network configuration, training loop, evaluation. Start here.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/feedforward/mnist/MLPMnistSingleLayerExample.java)

### MLPMnistTwoLayerExample.java

Extends the single-layer example with a second hidden layer. Demonstrates how adding depth affects training dynamics and accuracy on the same dataset.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/feedforward/mnist/MLPMnistTwoLayerExample.java)

### Feedforward examples directory

Contains additional classification examples (anomaly detection, multi-class iris classification, linear data classification) and regression examples. Browse the directory for the task closest to yours.

[Source directory](http://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/feedforward)

---

## Regression Examples

### RegressionMathFunctions.java

Trains a feedforward network to approximate several mathematical functions (sin, cos, x^2, etc.) from a single input value. The cleanest illustration of regression with `MultiLayerNetwork`: shows how to set up `LossFunction.MSE` and how to evaluate using `RegressionEvaluation`.

[Source](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/feedforward/regression/RegressionMathFunctions.java)

---

## CNN Examples (Image Classification)

### AnimalsClassification.java

Classifies images of four animal categories (bear, deer, duck, turtle) loaded from a local image directory. Can be run with either AlexNet or LeNet. Demonstrates `FileSplit`, `ParentPathLabelGenerator`, `ImageRecordReader`, and `RecordReaderDataSetIterator` — the standard image loading stack. To use your own image dataset, organize images into one subdirectory per class and point `FileSplit` at the root.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/convolution/AnimalsClassification.java)

### Transfer Learning with VGG16

The transfer learning examples show how to take a pretrained ImageNet model from the model zoo and adapt it to a new classification task by replacing the output layer:

- **FeaturizedPreSave.java** — runs the pretrained VGG16 up to a specified layer and saves the activations (features) to disk. Only needs to be run once.
- **FitFromFeaturized.java** — loads those saved features and trains a new classification head on them. Very fast because the expensive convolutional forward pass is precomputed.
- **EditLastLayerOthersFrozen.java** — demonstrates modifying VGG16 by freezing all layers except the final output layer and fine-tuning end-to-end.

[Transfer learning examples directory](https://github.com/eclipse/deeplearning4j-examples/tree/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/transferlearning/vgg16)

---

## RNN Examples (Sequence Data)

### BasicRNNExample.java

A minimal RNN that learns to reproduce a string of characters from a small alphabet. The simplest possible demonstration of a recurrent network in DL4J.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/basic/BasicRNNExample.java)

### GravesLSTMCharModellingExample.java

Trains an LSTM on the complete works of Shakespeare, character by character, then generates text in the style of Shakespeare. A famous example of character-level language modeling. The generated text quality improves noticeably over training epochs.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/character/GravesLSTMCharModellingExample.java)

### UCISequenceClassificationExample.java

Classifies time series into six categories (cyclic, uptrending, downtrending, etc.) using the UCI time series dataset. Shows how to load variable-length sequences, pad them to equal length, and use `RnnSequenceClassificationEvaluation`.

[Source](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/seqclassification/UCISequenceClassificationExample.java)

### Word2VecSentimentRNN.java

Sentiment analysis (positive/negative) on movie reviews using Word2Vec embeddings fed into an LSTM. Demonstrates combining pretrained word vectors with a recurrent classifier — a common pattern in NLP.

[Source](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/word2vecsentiment/Word2VecSentimentRNN.java)

### VideoClassificationExample.java

Combines convolutional layers (to process each frame), max pooling, dense layers, and LSTM layers (to capture temporal dynamics across frames) into a single ComputationGraph for video classification. A good example of multi-modal architectures.

[Source](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/recurrent/video/VideoClassificationExample.java)

---

## NLP Examples

### Word2VecRawTextExample.java

Trains a Word2Vec model from scratch on a raw text file and evaluates the resulting embedding space. Demonstrates `SentenceIterator`, `TokenizerFactory`, `Word2Vec.Builder`, and the `wordsNearest()` similarity API.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/word2vec/Word2VecRawTextExample.java)

### TSNEStandardExample.java

Loads pre-trained word vectors, runs t-SNE to reduce them to 2D, and saves coordinates to a CSV file for plotting. Shows that semantically similar words cluster together in the embedding space.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/tsne/TSNEStandardExample.java)

### ParagraphVectorsClassifierExample.java

Uses Paragraph Vectors (Doc2Vec) to classify documents. Each document is embedded as a fixed-length vector before classification.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/paragraphvectors/ParagraphVectorsClassifierExample.java)

---

## Saving and Loading Models

### SaveLoadMultiLayerNetwork.java / SaveLoadComputationGraph.java

These examples demonstrate serializing a trained network to a zip file and reloading it for inference or continued training using `ModelSerializer`. Essential for any production workflow.

[ComputationGraph source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/modelsaving/SaveLoadComputationGraph.java)

---

## Advanced Examples: Custom Layers and Loss Functions

### CustomLayerExample.java

Shows the minimum required to implement a custom layer: defining forward pass, backward pass, parameter initialization, and integrating with the DL4J builder API. Start here if you need a layer type that DL4J does not provide.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/customlayers/CustomLayerExample.java)

### CustomLossExample.java / CustomLossL1L2.java

Demonstrates implementing a custom loss function by extending `ILossFunction`. Includes both the loss computation and its gradient.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/misc/lossfunctions/CustomLossExample.java)

---

## Distributed Training on Spark

### MnistMLPExample.java

Trains a feedforward network on the MNIST dataset across a Spark cluster using `SparkDl4jMultiLayer`. Shows how to set up a `ParameterAveragingTrainingMaster` and adapt a single-node training loop for distributed use.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-spark-examples/dl4j-spark/src/main/java/org/deeplearning4j/mlp/MnistMLPExample.java)

### SparkLSTMCharacterExample.java

The Shakespeare LSTM text generation example, adapted for Spark. Useful for seeing how the Spark integration handles recurrent networks and time series data distribution.

[Source](http://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-spark-examples/dl4j-spark/src/main/java/org/deeplearning4j/rnn/SparkLSTMCharacterExample.java)

---

## ND4J Examples

ND4J is the tensor library DL4J is built on. The ND4J examples demonstrate creating and manipulating `INDArray` objects directly — useful if you need to implement custom preprocessing or mathematical operations outside the DL4J layer abstraction.

[ND4J examples directory](http://github.com/eclipse/deeplearning4j-examples/tree/master/nd4j-examples/src/main/java/org/nd4j/examples)

---

## Reinforcement Learning Examples (RL4J)

RL4J is DL4J's reinforcement learning library. Examples include agents learning to play Atari-style games and custom environments. The examples are in the `rl4j-examples` sub-module.

[RL4J examples directory](http://github.com/eclipse/deeplearning4j-examples/tree/master/rl4j-examples)

---

## Running Examples from the Command Line

After building with `mvn package`, you can run any example from the command line:

```bash
cd dl4j-examples
java -cp target/dl4j-examples-*-bin.jar \
    org.deeplearning4j.examples.feedforward.mnist.MLPMnistSingleLayerExample
```

Replace the class name with the example you want to run. The `-bin.jar` artifact produced by the Maven Shade plugin includes all dependencies.
