---
title: "Doc2Vec"
description: "Document vectors with ParagraphVectors (Doc2Vec) — training document embeddings and document similarity"
---

# Doc2Vec (ParagraphVectors)

Doc2Vec is an extension of Word2Vec that learns vector representations for entire documents in addition to individual words. In Deeplearning4j, the algorithm is implemented as `ParagraphVectors`, following the terminology from the original paper by Le and Mikolov. The core idea is to add a document-level token — the "paragraph vector" — to each training window alongside the word tokens. This document token is updated during training along with the word vectors, and at the end of training it encodes the semantic content of the entire document.

The result is a model that can:
- Return a vector for any training document given its label
- Infer a vector for a new, unseen document by running additional inference steps
- Find documents most similar to a query document using cosine similarity

---

## When to Use ParagraphVectors

Use `ParagraphVectors` instead of `Word2Vec` when:

- Your task requires a fixed-size vector representing a whole document, paragraph, or labeled text span
- You need to classify, cluster, or find similar documents
- You have labeled training data and want supervised document embeddings
- You want to infer vectors for new documents not seen during training

If you only need word-level embeddings (for use as features in an RNN or CNN), `Word2Vec` is simpler and sufficient.

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-nlp</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

---

## Core Concepts

### Labels

Every document in a `ParagraphVectors` training set is associated with one or more string labels. During training, each label gets its own vector (stored alongside word vectors in the lookup table). This is the key difference from `Word2Vec`: the label vectors are the document embeddings.

Labels can be:
- **Document IDs** — each document has a unique label; you learn one vector per document
- **Category labels** — multiple documents share the same label; the model learns one vector per category
- **Hierarchical labels** — a document can have multiple labels at once

### LabelAwareIterator

`ParagraphVectors` requires an iterator that returns not just sentences but also their associated labels. The `LabelAwareIterator` interface extends `SentenceIterator` with label awareness.

DL4J provides `LabelledDocument` as the unit returned by these iterators — a structure that holds the text and the list of labels for one document.

---

## Training ParagraphVectors

### Option 1 — Directory-Based Training (Files as Labels)

The simplest setup uses a directory where each subdirectory name is a label and each file in the subdirectory is a document with that label:

```
/data/labeled/
    positive/
        review001.txt
        review002.txt
    negative/
        review010.txt
        review011.txt
    neutral/
        review020.txt
```

```java
import org.deeplearning4j.models.paragraphvectors.ParagraphVectors;
import org.deeplearning4j.text.documentiterator.FileLabelAwareIterator;
import org.deeplearning4j.text.documentiterator.LabelAwareIterator;
import org.deeplearning4j.text.tokenization.tokenizer.preprocessor.CommonPreprocessor;
import org.deeplearning4j.text.tokenization.tokenizerfactory.DefaultTokenizerFactory;
import org.deeplearning4j.text.tokenization.tokenizerfactory.TokenizerFactory;

File labeledDataDir = new File("/data/labeled/");
LabelAwareIterator iterator = new FileLabelAwareIterator.Builder()
        .addSourceFolder(labeledDataDir)
        .build();

TokenizerFactory tokenizerFactory = new DefaultTokenizerFactory();
tokenizerFactory.setTokenPreProcessor(new CommonPreprocessor());

ParagraphVectors model = new ParagraphVectors.Builder()
        .minWordFrequency(1)
        .layerSize(100)
        .windowSize(5)
        .iterations(1)
        .epochs(1)
        .iterate(iterator)
        .tokenizerFactory(tokenizerFactory)
        .trainWordVectors(true)
        .build();

model.fit();
```

`trainWordVectors(true)` (the default) trains both word and document vectors simultaneously. Set it to `false` to only update document vectors, which is appropriate when you have pretrained word vectors you want to keep fixed.

### Option 2 — In-Memory Documents with Labels

When your documents are already in memory, use `BasicLabelAwareSentenceIterator` or supply documents via `CollectionLabelAwareIterator`:

```java
import org.deeplearning4j.text.documentiterator.LabelledDocument;

List<LabelledDocument> documents = new ArrayList<>();

LabelledDocument doc1 = new LabelledDocument();
doc1.setContent("The film was outstanding and deeply moving.");
doc1.addLabel("positive");
documents.add(doc1);

LabelledDocument doc2 = new LabelledDocument();
doc2.setContent("Terrible pacing and unconvincing acting throughout.");
doc2.addLabel("negative");
documents.add(doc2);

// Add more documents...

LabelAwareIterator iterator = new CollectionLabelAwareIterator(documents);
```

### Option 3 — UIMA-Based Label-Aware Iterator

For linguistically complex corpora where sentence segmentation matters:

```java
LabelAwareIterator iterator = LabelAwareUimaSentenceIterator.createWithPath(
        labeledDataDir.getAbsolutePath());
```

---

## ParagraphVectors.Builder Parameters

| Parameter | Method | Description |
|---|---|---|
| Minimum word frequency | `.minWordFrequency(int)` | Words below this count are excluded from the word vocabulary. Document label tokens are always kept. |
| Vector size | `.layerSize(int)` | Dimensionality of both word and document vectors. |
| Window size | `.windowSize(int)` | Context window size used during training. |
| Iterations | `.iterations(int)` | Number of updates per batch. |
| Epochs | `.epochs(int)` | Number of full passes through the training corpus. |
| Train word vectors | `.trainWordVectors(boolean)` | Whether to update word vectors alongside document vectors. |
| Labels | `.labels(List<String>)` | Explicit list of label names when not using `LabelAwareIterator`. |
| Learning rate | `.learningRate(double)` | Initial SGD learning rate. |
| Minimum learning rate | `.minLearningRate(double)` | Learning rate floor. |
| Sampling threshold | `.sampling(double)` | Downsampling threshold for frequent words. Values around 1e-5 work well for most corpora. |

---

## Querying Document Vectors

### Get the Vector for a Training Label

```java
// Returns INDArray of shape [1, layerSize]
INDArray positiveVec = model.getLookupTable().vector("positive");
INDArray negativeVec = model.getLookupTable().vector("negative");
```

### Similarity Between Two Labels

```java
double sim = model.similarity("positive", "negative");
System.out.println("Positive vs Negative similarity: " + sim);
// A well-trained sentiment model will return a low value here
```

### Nearest Labels to a Query Label

```java
Collection<String> nearest = model.nearestLabels("positive", 3);
System.out.println(nearest);
```

### Words Nearest to a Label

Because word and document vectors live in the same space, you can find which words are most characteristic of a label:

```java
Collection<String> topWords = model.wordsNearest("positive", 10);
System.out.println("Most characteristic words for 'positive': " + topWords);
```

---

## Inferring Vectors for New Documents

Training learns vectors for documents seen during training. To get a vector for a new, unseen document, use `inferVector`:

```java
String newDocument = "An exceptional performance that left the audience speechless.";

// The model runs additional gradient descent steps to find the best vector
// for this document given the fixed word vectors
INDArray inferred = model.inferVector(newDocument);
```

You can also pass a list of tokens directly:

```java
List<String> tokens = Arrays.asList("exceptional", "performance", "audience");
INDArray inferred = model.inferVector(tokens);
```

`inferVector` is more expensive than a lookup because it requires additional optimization steps. The number of inference iterations and the learning rate during inference can be controlled:

```java
// Inference with explicit parameters: 100 iterations, learning rate 0.02
INDArray inferred = model.inferVector(newDocument, 0.02, 0.001, 100);
```

### Classifying a New Document

After inferring a vector for a new document, find the nearest training label:

```java
INDArray docVector = model.inferVector("Absolutely wonderful film.");
Collection<String> nearestLabels = model.nearestLabels(docVector, 1);
String predictedLabel = nearestLabels.iterator().next();
System.out.println("Predicted label: " + predictedLabel);  // "positive"
```

---

## Document Similarity Search

To find training documents most similar to a query document:

```java
// Both labels and words live in the same lookup table
// Nearest neighbors searches over all tokens including label tokens
INDArray queryVec = model.inferVector("A slow and tedious movie.");

// Find nearest label (classification)
Collection<String> label = model.nearestLabels(queryVec, 1);

// Find nearest label with similarity score
double sim = model.similarityToLabel(queryVec, "negative");
```

---

## Saving and Loading

### Save

```java
import org.deeplearning4j.models.embeddings.loader.WordVectorSerializer;

WordVectorSerializer.writeParagraphVectors(model, "/models/doc2vec.bin");
```

### Load

```java
ParagraphVectors loaded = WordVectorSerializer.readParagraphVectors("/models/doc2vec.bin");

// After loading, attach a tokenizer factory before calling inferVector
loaded.setTokenizerFactory(tokenizerFactory);
```

Always re-attach a `TokenizerFactory` after loading when you intend to call `inferVector`, because the tokenizer factory is not serialized with the model.

---

## Full Classification Example

```java
// Training
File trainDir = new File("/data/sentiment/train/");
LabelAwareIterator trainIter = new FileLabelAwareIterator.Builder()
        .addSourceFolder(trainDir)
        .build();

TokenizerFactory tf = new DefaultTokenizerFactory();
tf.setTokenPreProcessor(new CommonPreprocessor());

ParagraphVectors model = new ParagraphVectors.Builder()
        .minWordFrequency(2)
        .layerSize(150)
        .windowSize(5)
        .epochs(3)
        .iterate(trainIter)
        .tokenizerFactory(tf)
        .trainWordVectors(true)
        .build();

model.fit();

WordVectorSerializer.writeParagraphVectors(model, "/models/sentiment.bin");

// Inference on new data
ParagraphVectors classifier = WordVectorSerializer.readParagraphVectors("/models/sentiment.bin");
classifier.setTokenizerFactory(tf);

String review = "A masterpiece. The direction is flawless and the acting superb.";
INDArray vec = classifier.inferVector(review);
Collection<String> labels = classifier.nearestLabels(vec, 1);
System.out.println("Predicted sentiment: " + labels.iterator().next());
```

---

## Further Reading

- [Word2Vec](word2vec.md) — word-level embeddings, which ParagraphVectors builds on
- [Sentence Iterators](sentence-iterator.md) — general-purpose text input
- [Tokenization](tokenization.md) — tokenizer factories and preprocessors
- Le and Mikolov, [Distributed Representations of Sentences and Documents](https://cs.stanford.edu/~quocle/paragraph_vector.pdf) (2014)
- [DL4J ParagraphVectors classifier example](https://github.com/eclipse/deeplearning4j-examples/blob/master/dl4j-examples/src/main/java/org/deeplearning4j/examples/nlp/paragraphvectors/ParagraphVectorsClassifierExample.java)
