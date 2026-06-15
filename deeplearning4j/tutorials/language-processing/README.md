---
title: "NLP Overview"
description: "Natural language processing in Deeplearning4j — Word2Vec, Doc2Vec, tokenization, and text processing pipeline"
---

# NLP Overview

Deeplearning4j provides a focused set of natural language processing (NLP) tools designed for training and evaluating neural word embeddings and document embeddings on the JVM. The toolkit is not a full NLP pipeline in the style of Stanford CoreNLP or spaCy — it does not include dependency parsers, named-entity recognizers, or syntactic analyzers out of the box — but it covers the preprocessing and representation learning steps needed to feed text into deep learning models.

The core supported algorithms are:

- **Word2Vec** — unsupervised learning of dense word vectors via Skip-Gram or CBOW objectives
- **Doc2Vec (ParagraphVectors)** — extension of Word2Vec that learns vector representations for entire documents or labeled text spans
- **GloVe-compatible loading** — load pretrained GloVe vectors and use them as word embeddings

For heavier linguistic preprocessing (sentence boundary detection, part-of-speech tagging, lemmatization), DL4J integrates with [Apache UIMA](https://uima.apache.org/) and the ClearTK framework. UIMA-based components are available but are optional — for most Word2Vec and Doc2Vec workflows, the built-in iterators and tokenizers are sufficient.

---

## The Text Processing Pipeline

Every NLP workflow in DL4J follows the same three-stage pipeline:

```
Raw text
   |
   v
SentenceIterator          -- produces one sentence (string) at a time
   |
   v
TokenizerFactory          -- splits each sentence into tokens (words)
   |
   v
Word2Vec / ParagraphVectors  -- learns vector representations
```

Understanding each stage makes it straightforward to swap components or build custom ones.

---

## Stage 1: SentenceIterator

A `SentenceIterator` feeds raw text into the training process one sentence at a time. "Sentence" here means any unit of text that should be processed as a coherent context window — it might be an actual sentence, a tweet, a paragraph, or a document depending on your task.

DL4J ships several implementations:

| Class | Input source |
|---|---|
| `BasicLineIterator` | Reads a plain text file; one line = one sentence |
| `LineSentenceIterator` | Like `BasicLineIterator`; accepts a `File` and allows a preprocessor |
| `CollectionSentenceIterator` | Iterates over a `Collection<String>` already in memory |
| `FileSentenceIterator` | Recursively walks a directory and returns lines |
| `UimaSentenceIterator` | Uses OpenNLP / ClearTK for linguistically correct sentence segmentation |

Minimal example using a text file:

```java
SentenceIterator iterator = new BasicLineIterator("/path/to/corpus.txt");
```

With a lowercase preprocessor:

```java
SentenceIterator iterator = new LineSentenceIterator(new File("/path/to/corpus.txt"));
iterator.setPreProcessor(sentence -> sentence.toLowerCase());
```

For in-memory data such as a list of tweets:

```java
List<String> tweets = Arrays.asList("Hello world", "DL4J is great", ...);
SentenceIterator iterator = new CollectionSentenceIterator(tweets);
```

See [Sentence Iterators](sentence-iterator.md) for the full reference including custom implementations.

---

## Stage 2: TokenizerFactory

A `TokenizerFactory` converts each sentence string into a sequence of tokens (typically words). It is stateless with respect to the vocabulary — vocabulary construction happens later inside the model.

The standard choice for most tasks:

```java
TokenizerFactory tokenizerFactory = new DefaultTokenizerFactory();
tokenizerFactory.setTokenPreProcessor(new CommonPreprocessor());
```

`CommonPreprocessor` applies lowercase conversion and strips punctuation, which is the most common normalization step before word embedding training.

Alternative factories:

- `NGramTokenizerFactory` — wraps another factory and produces n-gram tokens in addition to unigrams
- `UimaTokenizerFactory` — linguistically accurate tokenization with stemming and POS tagging via UIMA

See [Tokenization](tokenization.md) for details on preprocessors and custom tokenizers.

---

## Stage 3: Word2Vec or ParagraphVectors

With an iterator and a tokenizer factory in hand, you wire them into a model builder:

```java
Word2Vec model = new Word2Vec.Builder()
        .minWordFrequency(5)
        .layerSize(100)
        .windowSize(5)
        .iterate(iterator)
        .tokenizerFactory(tokenizerFactory)
        .build();

model.fit();
```

The model builds a `VocabCache` internally (filtering out words below `minWordFrequency`), then trains word vectors. After `fit()` completes, the model can answer nearest-neighbor queries and produce vector representations for any in-vocabulary word.

---

## Key Classes at a Glance

| Class / Interface | Package | Role |
|---|---|---|
| `SentenceIterator` | `org.deeplearning4j.text.sentenceiterator` | Produces raw sentence strings |
| `TokenizerFactory` | `org.deeplearning4j.text.tokenization.tokenizerfactory` | Creates `Tokenizer` instances per sentence |
| `Tokenizer` | `org.deeplearning4j.text.tokenization.tokenizer` | Splits one sentence into tokens |
| `TokenPreProcess` | `org.deeplearning4j.text.tokenization.tokenizer` | Normalizes individual tokens |
| `VocabCache` | `org.deeplearning4j.models.word2vec.wordstore` | Stores vocabulary, word counts, and indices |
| `Word2Vec` | `org.deeplearning4j.models.word2vec` | Trains and queries word embeddings |
| `ParagraphVectors` | `org.deeplearning4j.models.paragraphvectors` | Trains document + word embeddings |
| `WordVectorSerializer` | `org.deeplearning4j.models.embeddings.loader` | Saves and loads models and pretrained vectors |
| `InMemoryLookupTable` | `org.deeplearning4j.models.embeddings.inmemory` | Stores the weight matrix (syn0/syn1) |

---

## Choosing the Right Component

**Use Word2Vec when:**
- Your goal is word-level similarity, analogy reasoning, or producing word embeddings as input features for a downstream neural network
- You have a large text corpus and want unsupervised representation learning
- You want to load pretrained embeddings (GloVe, Google News vectors) and use them with DL4J tooling

**Use ParagraphVectors (Doc2Vec) when:**
- You need vector representations for whole documents or labeled text categories
- You are doing document classification, clustering, or similarity search at the document level
- You have labeled training data and want to jointly learn document and word vectors

**Use UimaSentenceIterator / UimaTokenizerFactory when:**
- Sentence boundaries in your corpus are ambiguous (e.g., prose text with abbreviations)
- You need linguistically accurate tokenization — for example, separating "isn't" into "is" and "n't"
- You are working with languages or domains where simple whitespace splitting is inadequate

For most English-language corpora where sentences are already one-per-line (logs, tweets, Wikipedia sentence-split dumps), `BasicLineIterator` plus `DefaultTokenizerFactory` with `CommonPreprocessor` is the right starting point. Add UIMA only when the simpler tools produce noticeable quality problems.

---

## Maven Dependency

All NLP classes live in the `deeplearning4j-nlp` artifact:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-nlp</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

UIMA-based components require the additional `deeplearning4j-nlp-uima` artifact:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-nlp-uima</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

---

## End-to-End Example

The following snippet shows the complete pipeline from file to trained Word2Vec model:

```java
import org.deeplearning4j.models.word2vec.Word2Vec;
import org.deeplearning4j.text.sentenceiterator.BasicLineIterator;
import org.deeplearning4j.text.sentenceiterator.SentenceIterator;
import org.deeplearning4j.text.tokenization.tokenizer.preprocessor.CommonPreprocessor;
import org.deeplearning4j.text.tokenization.tokenizerfactory.DefaultTokenizerFactory;
import org.deeplearning4j.text.tokenization.tokenizerfactory.TokenizerFactory;
import org.deeplearning4j.models.embeddings.loader.WordVectorSerializer;

// 1. Load sentences
SentenceIterator iterator = new BasicLineIterator("/data/corpus.txt");

// 2. Configure tokenizer
TokenizerFactory tokenizerFactory = new DefaultTokenizerFactory();
tokenizerFactory.setTokenPreProcessor(new CommonPreprocessor());

// 3. Build and train Word2Vec
Word2Vec model = new Word2Vec.Builder()
        .minWordFrequency(5)
        .layerSize(100)
        .windowSize(5)
        .iterations(1)
        .seed(42)
        .iterate(iterator)
        .tokenizerFactory(tokenizerFactory)
        .build();

model.fit();

// 4. Query the model
Collection<String> nearest = model.wordsNearest("deep", 10);
System.out.println("Nearest to 'deep': " + nearest);

double similarity = model.similarity("neural", "network");
System.out.println("Similarity neural/network: " + similarity);

// 5. Save
WordVectorSerializer.writeWord2VecModel(model, "/models/word2vec.bin");
```

---

## Further Reading

- [Word2Vec](word2vec.md) — detailed guide to training and querying word embeddings
- [Doc2Vec / ParagraphVectors](doc2vec.md) — document-level embeddings
- [Sentence Iterators](sentence-iterator.md) — all iterator types and custom iterators
- [Tokenization](tokenization.md) — tokenizer factories, preprocessors, and custom tokenizers
- [Vocabulary Cache](vocabulary-cache.md) — how the vocabulary is built and managed
