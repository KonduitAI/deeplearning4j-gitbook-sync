---
title: "Word2Vec"
description: "Word2Vec in Deeplearning4j — Skip-Gram, CBOW, training word vectors, and using pretrained embeddings"
---

# Word2Vec

Word2Vec is a family of shallow neural network algorithms that learn dense vector representations of words from raw text. Given a large corpus, it produces a lookup table — the **word embedding matrix** — where each row is a low-dimensional vector encoding the distributional meaning of one word. Words that appear in similar contexts end up geometrically close in vector space.

Deeplearning4j ships a distributed, multi-threaded implementation of Word2Vec for Java and Scala. It supports training from scratch on raw text files as well as loading pretrained embeddings in the Google binary format, C text format, and GloVe text format.

---

## Skip-Gram vs CBOW

Word2Vec offers two training objectives:

**Skip-Gram** (default in DL4J) predicts the surrounding context words given a center word. For each position in the text, the model asks: "given this word, what words tend to appear nearby?" Skip-Gram handles rare words better because each training example focuses on one center word, giving rare words many training updates relative to their frequency.

**CBOW (Continuous Bag of Words)** inverts the objective: given the surrounding context words, predict the center word. CBOW is faster to train and works well on larger corpora where frequent words dominate.

Both variants learn the same weight matrix. The practical difference:
- Skip-Gram: better for smaller corpora or corpora with many infrequent terms
- CBOW: faster convergence on very large corpora

DL4J's implementation uses negative sampling to approximate the softmax denominator efficiently.

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

## Training Word2Vec

### Step 1 — Create a SentenceIterator

The `SentenceIterator` feeds sentences from your corpus into the training loop one at a time.

```java
// One sentence per line in a plain text file
SentenceIterator iterator = new BasicLineIterator("/data/corpus.txt");
```

For a file where sentence boundaries must be detected linguistically, use `UimaSentenceIterator`:

```java
SentenceIterator iterator = UimaSentenceIterator.create("/data/documents/");
```

### Step 2 — Configure a TokenizerFactory

The `TokenizerFactory` splits each sentence string into individual word tokens.

```java
TokenizerFactory tokenizerFactory = new DefaultTokenizerFactory();
tokenizerFactory.setTokenPreProcessor(new CommonPreprocessor());
```

`CommonPreprocessor` lowercases tokens and strips non-alphanumeric characters. This single preprocessor handles most English corpora. See [Tokenization](tokenization.md) for alternatives.

### Step 3 — Build and Train the Model

```java
Word2Vec model = new Word2Vec.Builder()
        .minWordFrequency(5)
        .layerSize(100)
        .windowSize(5)
        .iterations(1)
        .epochs(1)
        .seed(42)
        .learningRate(0.025)
        .minLearningRate(1e-4)
        .negativeSample(10)
        .iterate(iterator)
        .tokenizerFactory(tokenizerFactory)
        .build();

model.fit();
```

---

## Key Builder Parameters

| Parameter | Method | Default | Effect |
|---|---|---|---|
| Minimum word count | `.minWordFrequency(int)` | 5 | Words appearing fewer times than this are excluded from the vocabulary. Raise this on very large corpora to reduce noise and memory. |
| Vector dimensionality | `.layerSize(int)` | 100 | Number of dimensions in each word vector. Larger vectors capture more nuance but require more memory and training time. Common choices: 100, 200, 300. |
| Context window | `.windowSize(int)` | 5 | Number of words to the left and right of the center word used as context. Smaller windows capture syntactic relations; larger windows capture topical/semantic relations. |
| Training iterations | `.iterations(int)` | 1 | Number of passes through each batch. Typically kept at 1; increase `epochs` to do multiple full corpus passes instead. |
| Epochs | `.epochs(int)` | 1 | Number of complete passes through the corpus. More epochs improve quality at the cost of training time. |
| Learning rate | `.learningRate(double)` | 0.025 | Initial step size for SGD updates. |
| Minimum learning rate | `.minLearningRate(double)` | 1e-4 | Learning rate floor; prevents the rate from decaying to zero. |
| Negative samples | `.negativeSample(int)` | 10 | Number of noise words sampled per positive example for negative sampling. Values 5–20 are common. |
| Random seed | `.seed(long)` | — | Controls the random number generator for reproducibility. |
| Batch size | `.batchSize(int)` | 512 | Words processed per update step. Larger batches improve throughput on multi-core machines. |
| Use AdaGrad | `.useAdaGrad(boolean)` | false | Enables per-parameter adaptive gradient updates. Useful for very sparse vocabularies. |
| Large model | `.hugeModelExpected(boolean)` | false | When `true`, periodically truncates low-frequency words from the vocabulary during the build phase to avoid OOM errors on very large corpora. |

---

## Querying the Trained Model

### Nearest Neighbors

`wordsNearest` returns the N words closest to a query word in vector space by cosine similarity:

```java
Collection<String> nearest = model.wordsNearest("king", 10);
System.out.println(nearest);
// [queen, prince, monarch, emperor, throne, ruler, dynasty, crown, princess, reign]
```

### Cosine Similarity

```java
double sim = model.similarity("cat", "dog");
System.out.println(sim);  // e.g. 0.82
```

Values range from -1 to 1. A value near 1 means the model considers the two words highly similar in context.

### Word Vector Arithmetic

Word2Vec supports analogy reasoning via vector arithmetic. The canonical example is `king - man + woman ≈ queen`:

```java
// Positive words contribute their vectors; negative words subtract
Collection<String> result = model.wordsNearest(
        Arrays.asList("king", "woman"),   // positive
        Arrays.asList("man"),             // negative
        5                                  // top N
);
System.out.println(result);
// [queen, princess, monarch, duchess, empress]
```

### Getting a Raw Vector

```java
// Returns an INDArray of shape [1, layerSize]
INDArray vector = model.getWordVectorMatrix("neural");

// Returns a double[] of length layerSize
double[] vectorArray = model.getWordVector("neural");
```

If the word is not in the vocabulary, `getWordVectorMatrix` returns a zero vector and `wordsNearest` ignores it.

### Checking Vocabulary Membership

```java
boolean inVocab = model.hasWord("deeplearning");

// Iterate over all vocabulary words
Collection<String> vocab = model.vocab().words();
```

---

## Saving and Loading Models

### Save in DL4J Binary Format

```java
WordVectorSerializer.writeWord2VecModel(model, "/models/word2vec.bin");
```

This format preserves the full model state including vocabulary, weights, and configuration. Prefer this format when you intend to reload the model for further training or inference in DL4J.

### Load DL4J Binary Format

```java
Word2Vec loaded = WordVectorSerializer.readWord2VecModel("/models/word2vec.bin");
```

### Save as Plain Text (One Word Per Line + Vector)

```java
WordVectorSerializer.writeWordVectors(model, "/models/word2vec.txt");
```

The text format is a plain file where each line is a word followed by space-separated floating point numbers. This format is broadly compatible with other tools.

### Continue Training (Uptraining)

You can continue training an existing model on new data:

```java
Word2Vec existing = WordVectorSerializer.readWord2VecModel("/models/word2vec.bin");
existing.setTokenizerFactory(tokenizerFactory);
existing.setSentenceIterator(newIterator);
existing.fit();
```

This adds new vocabulary and updates existing word vectors. Useful when your corpus grows incrementally.

---

## Loading Pretrained Embeddings

### Google News Vectors (Binary Format)

Google's pretrained model covers 3 million words with 300-dimensional vectors trained on 100 billion words from Google News:

```java
File modelFile = new File("/models/GoogleNews-vectors-negative300.bin.gz");
Word2Vec pretrained = WordVectorSerializer.readWord2VecModel(modelFile);
```

This model requires roughly 4–8 GB of heap space depending on JVM version. Set your heap with `-Xmx10g` when loading it.

### GloVe Vectors (Text Format)

GloVe vectors are distributed as a plain text file with the same format as the Word2Vec text format:

```java
WordVectors glove = WordVectorSerializer.loadTxtVectors(
        new File("/models/glove.6B.100d.txt"));

// Query exactly the same way as a trained model
Collection<String> nearest = glove.wordsNearest("science", 5);
double sim = glove.similarity("paris", "france");
```

### C Text Format (Gensim / Original C Tool)

Models exported by the original C Word2Vec tool or by Gensim in text format can be loaded directly:

```java
Word2Vec model = WordVectorSerializer.readWord2VecModel(
        new File("/models/vectors.txt"));
```

For compressed files in the Google binary gzip format, pass `true` as the second argument to indicate binary:

```java
Word2Vec model = WordVectorSerializer.readWord2VecModel(
        new File("/models/vectors.bin.gz"), true);
```

---

## Using Word Vectors as Input Features

A common pattern is to train or load Word2Vec, then feed word vectors into a recurrent or convolutional neural network:

```java
// Retrieve the lookup table for use as an embedding layer initializer
WeightLookupTable lookupTable = model.lookupTable();

// Get the embedding weight matrix (INDArray of shape [vocabSize, layerSize])
INDArray syn0 = ((InMemoryLookupTable) lookupTable).getSyn0();

// Or get a single word's vector
INDArray wordVec = model.getWordVectorMatrix("example");
```

When building a `ComputationGraph` or `MultiLayerNetwork` with an embedding layer, you can initialize the embedding weights from the pretrained vectors to speed up convergence significantly.

---

## Visualization with t-SNE

Visualizing the embedding space with t-SNE helps verify that semantically related words cluster together:

```java
Pair<InMemoryLookupTable, VocabCache> vectors =
        WordVectorSerializer.loadTxt(new File("/models/word2vec.txt"));

INDArray weights = vectors.getFirst().getSyn0();
List<String> words = new ArrayList<>();
for (int i = 0; i < vectors.getSecond().numWords(); i++) {
    words.add(vectors.getSecond().wordAtIndex(i));
}

BarnesHutTsne tsne = new BarnesHutTsne.Builder()
        .setMaxIter(1000)
        .theta(0.5)
        .normalize(false)
        .learningRate(500)
        .build();

tsne.fit(weights);
tsne.saveAsFile(words, "target/tsne-coords.csv");
```

Plot the resulting CSV with any visualization tool that accepts 2D coordinates.

---

## Troubleshooting

**Words are missing from the model**

Raise `minWordFrequency` and check whether the missing words actually appear with sufficient frequency in your corpus. Also check that your `TokenizerFactory` and preprocessor are not stripping them.

**Training is very slow**

- Check that each call to your `SentenceIterator.nextSentence()` returns a single sentence, not an entire document. Word2Vec gathers co-occurrence statistics within sentence boundaries. Loading an entire document as one "sentence" forces O(N^2) skip-gram calculations and degrades quality.
- Increase `batchSize` to better utilize available CPU cores.
- Set `iterations(1)` and use `epochs` for multiple passes rather than high iterations.

**StackOverflowError during training**

This is usually caused by ehcache disk store files accumulating in the working directory. Stop training, delete the directories named `ehcache_auto_created*`, and restart.

**Out of memory loading pretrained models**

Large models like Google News vectors require significant heap. Launch the JVM with `-Xmx10g` or higher. If you only need a subset of the vocabulary, consider filtering the file to the words you need before loading.

**Normalization mismatches**

Some query methods like `wordsNearest` use L2-normalized vectors internally, while raw `getWordVectorMatrix` returns un-normalized vectors. Mixing normalized and un-normalized vectors when computing custom similarity scores produces incorrect results. Use `model.getWordVectorMatrixNormalized("word")` if you need a normalized vector for manual cosine computation.

---

## Further Reading

- [Doc2Vec / ParagraphVectors](doc2vec.md) — extend Word2Vec to document-level representations
- [Sentence Iterators](sentence-iterator.md) — input pipeline for your corpus
- [Tokenization](tokenization.md) — tokenizer factories and preprocessors
- [Vocabulary Cache](vocabulary-cache.md) — how vocabulary is built and stored
- Mikolov et al., [Efficient Estimation of Word Representations in Vector Space](https://arxiv.org/abs/1301.3781) (2013)
- Mikolov et al., [Distributed Representations of Words and Phrases and their Compositionality](https://arxiv.org/abs/1310.4546) (2013)
