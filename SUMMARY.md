# Table of contents

* [Eclipse DeepLearning4J](README.md)

## Getting Started

* [Quickstart](getting-started/quickstart.md)
* [Tutorials](getting-started/tutorials/README.md)
  * [Quickstart with MNIST](getting-started/tutorials/quickstart-with-mnist.md)
  * [MultiLayerNetwork And ComputationGraph](getting-started/tutorials/multilayernetwork-and-computationgraph.md)
  * [Logistic Regression](getting-started/tutorials/logistic-regression.md)
  * [Built-in Data Iterators](getting-started/tutorials/built-in-data-iterators.md)
  * [Feed Forward Networks](getting-started/tutorials/feed-forward-networks.md)
  * [Basic Autoencoder](getting-started/tutorials/basic-autoencoder.md)
  * [Advanced Autoencoder](getting-started/tutorials/advanced-autoencoder.md)
  * [Convolutional Networks](getting-started/tutorials/convolutional-networks.md)
  * [Recurrent Networks](getting-started/tutorials/recurrent-networks.md)
  * [Early Stopping](getting-started/tutorials/early-stopping.md)
  * [Layers and Preprocessors](getting-started/tutorials/layers-and-preprocessors.md)
  * [Hyperparameter Optimization](getting-started/tutorials/hyperparameter-optimization.md)
  * [Using Multiple GPUs](getting-started/tutorials/using-multiple-gpus.md)
  * [Clinical Time Series LSTM](getting-started/tutorials/clinical-time-series-lstm.md)
  * [Sea Temperature Convolutional LSTM](getting-started/tutorials/sea-temperature-convolutional-lstm.md)
  * [Sea Temperature Convolutional LSTM 2](getting-started/tutorials/sea-temperature-convolutional-lstm-example-2.md)
  * [Instacart Multitask Example](getting-started/tutorials/instacart-multitask-example.md)
  * [Instacart Single Task Example](getting-started/tutorials/instacart-single-task-example.md)
  * [Cloud Detection Example](getting-started/tutorials/cloud-detection-example.md)
* [Core Concepts](getting-started/core-concepts.md)
* [Cheat Sheet](getting-started/cheat-sheet.md)
* [Examples Tour](getting-started/examples-tour.md)
* [Deep Learning Beginners](getting-started/beginners.md)
* [Build from Source](getting-started/build-from-source.md)
* [Contribute](getting-started/contribute/README.md)
  * [Eclipse Contributors](getting-started/contribute/eclipse-contributors.md)
* [Benchmark Guide](getting-started/benchmark.md)
* [About](getting-started/about.md)
* [Release Notes](getting-started/release-notes.md)

## Configuration <a id="config"></a>

* [Backends](config/backends/README.md)
  * [CPU and AVX](config/backends/cpu.md)
  * [cuDNN](config/backends/config-cudnn.md)
  * [Performance Issues](config/backends/performance-issues.md)
* [Memory Management](config/config-memory/README.md)
  * [Memory Workspaces](config/config-memory/config-workspaces.md)
* [Snapshots](config/config-snapshots.md)
* [Maven](config/maven.md)
* [SBT, Gradle, & Others](config/buildtools.md)

## Models

* [Autoencoders](models/autoencoders.md)
* [Multilayer Network](models/multilayernetwork.md)
* [Computation Graph](models/computationgraph.md)
* [Convolutional Neural Network](models/convolutional.md)
* [Recurrent Neural Network](models/recurrent.md)
* [Layers](models/layers.md)
* [Vertices](models/vertices.md)
* [Iterators](models/iterators.md)
* [Listeners](models/listeners.md)
* [Custom Layers](models/custom-layer.md)
* [Model Persistence](models/model-persistence.md)
* [Activations](models/activations.md)
* [Updaters](models/updaters.md)

## Model Zoo

* [Overview](model-zoo/overview.md)
* [Zoo Models](model-zoo/zoo-models.md)

## ND4J

* [Overview](nd4j/overview.md)
* [Quickstart](nd4j/quickstart.md)
* [Basics](nd4j/basics.md)
* [Elementwise Operations](nd4j/elementwise.md)
* [Matrix Manipulation](nd4j/matrix-manipulation.md)
* [Syntax](nd4j/syntax.md)
* [Tensors](nd4j/tensor.md)

## SAMEDIFF

* [Importing TensorFlow models](samediff/importing-tensorflow.md)
* [Variables](samediff/variables.md)
* [Ops](samediff/ops.md)
* [Adding Ops](samediff/adding-ops.md)

## Tuning & Training

* [Evaluation](tuning-and-training/evaluation.md)
* [Visualization](tuning-and-training/visualization.md)
* [Trouble Shooting](tuning-and-training/troubleshooting-training.md)
* [Early Stopping](tuning-and-training/early-stopping.md)
* [t-SNE Visualization](tuning-and-training/tsne-visualization.md)
* [Transfer Learning](tuning-and-training/transfer-learning.md)

## DISTRIBUTED DEEP LEARNING

* [Introduction/Getting Started](distributed-deep-learning/intro.md)
* [Technical Explanation](distributed-deep-learning/technicalref.md)
* [Spark Guide](distributed-deep-learning/howto.md)
* [Spark Data Pipelines Guide](distributed-deep-learning/data-howto.md)
* [API Reference](distributed-deep-learning/apiref.md)
* [Parameter Server](distributed-deep-learning/parameter-server.md)

## Keras Import

* [Overview](keras-import/overview.md)
* [Get Started](keras-import/get-started.md)
* [Supported Features](keras-import/supported-features/README.md)
  * [Activations](keras-import/supported-features/activations.md)
  * [Losses](keras-import/supported-features/losses.md)
  * [Regularizers](keras-import/supported-features/regularizers.md)
  * [Initializers](keras-import/supported-features/initializers.md)
  * [Constraints](keras-import/supported-features/constraints.md)
  * [Optimizers](keras-import/supported-features/optimizers.md)
* [Functional Model](keras-import/model-functional.md)
* [Sequential Model](keras-import/model-sequential.md)
* [API Reference](keras-import/api-reference/README.md)
  * [Core Layers](keras-import/api-reference/core-layers.md)
  * [Convolutional Layers](keras-import/api-reference/convolutional-layers.md)
  * [Embedding Layers](keras-import/api-reference/embedding-layers.md)
  * [Local Layers](keras-import/api-reference/local-layers.md)
  * [Noise Layers](keras-import/api-reference/noise-layers.md)
  * [Normalization Layers](keras-import/api-reference/normalization-layers.md)
  * [Pooling Layers](keras-import/api-reference/pooling-layers.md)
  * [Recurrent Layers](keras-import/api-reference/recurrent-layers.md)
  * [Wrapper Layers](keras-import/api-reference/wrapper-layers.md)
  * [Advanced Activations](keras-import/api-reference/advanced-activations.md)

## Arbiter

* [Overview](arbiter/overview.md)
* [Layer Spaces](arbiter/layer-spaces.md)
* [Parameter Spaces](arbiter/parameter-spaces.md)

## Datavec

* [Overview](datavec/overview.md)
* [Records](datavec/records.md)
* [Reductions](datavec/reductions.md)
* [Schema](datavec/schema.md)
* [Serialization](datavec/serialization.md)
* [Transforms](datavec/transforms.md)
* [Analysis](datavec/analysis.md)
* [Readers](datavec/readers.md)
* [Conditions](datavec/conditions.md)
* [Executors](datavec/executors.md)
* [Filters](datavec/filters.md)
* [Operations](datavec/operations.md)
* [Normalization](datavec/normalization.md)
* [Visualization](datavec/visualization.md)

## Language Processing

* [Overview](language-processing/overview.md)
* [Word2Vec](language-processing/word2vec.md)
* [Doc2Vec](language-processing/doc2vec.md)
* [Sentence Iteration](language-processing/sentence-iterator.md)
* [Tokenization](language-processing/tokenization.md)
* [Vocabulary Cache](language-processing/vocabulary-cache.md)

## Mobile \(Android\) <a id="android"></a>

* [Setup](android/setup.md)
* [Tutorial: First Steps](android/first-steps.md)
* [Tutorial: Classifier](android/linear-classifier.md)
* [Tutorial: Image Classifier](android/image-classification.md)
* [FAQ](faq.md)
* [Press](press.md)
* [Support](support.md)
* [Why Deep Learning?](why-deep-learning.md)

