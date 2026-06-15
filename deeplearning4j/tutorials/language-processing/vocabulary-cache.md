---
title: "Vocabulary Cache"
description: "VocabCache in Deeplearning4j — how vocabulary is stored, InMemoryLookupCache, and vocabulary management"
---

# Vocabulary Cache

The vocabulary cache (`VocabCache`) is the central data structure for NLP in Deeplearning4j. It holds every piece of metadata about the words and tokens encountered during corpus processing: which tokens exist, how often each occurred, the index that maps each word to a row in the embedding matrix, and the inverse-document frequency statistics used by TF-IDF.

You rarely construct or manipulate a `VocabCache` directly — Word2Vec and ParagraphVectors build and manage one internally. Understanding the vocabulary cache is useful when:

- You need to inspect the vocabulary after training
- You want to integrate DL4J embeddings with external tooling
- You are implementing a custom NLP component
- You need to understand why certain words are or are not in the trained model

---

## Tokens vs Vocabulary Words

The vocabulary cache maintains a strict distinction between **tokens** and **vocab words**:

- A **token** is any string that the tokenizer has seen at least once. All tokens are tracked in the cache, regardless of frequency.
- A **vocab word** is a token that has met the minimum frequency threshold (`minWordFrequency`). Only vocab words get rows in the embedding matrix and participate in Word2Vec training.

This two-level design means you can query the cache to understand how many times a word appeared (even if it was too rare to be trained on) without confusing it with the words that actually influence the model.

---

## The VocabCache Interface

```java
public interface VocabCache<T extends SequenceElement> extends Serializable {

    // Token-level operations
    void addToken(T token);
    T tokenFor(String word);
    boolean hasToken(String word);
    Collection<String> words();

    // Vocab word operations
    void addWordToIndex(int index, String word);
    void putVocabWord(String word);
    boolean containsWord(String word);
    T wordFor(String word);
    String wordAtIndex(int index);
    int indexOf(String word);

    // Frequency and statistics
    int docAppearedIn(String word);
    void incrementDocCount(String word, long delta);
    void setCountForDoc(String word, long count);
    long totalWordOccurrences();
    double totalNumberOfDocs();
    int numWords();

    // Inverse document frequency
    void incrementTotalDocCount();
    double idf(String word);
    double documentFrequency(String word);
}
```

The most commonly used methods in practice are `containsWord`, `wordFor`, `indexOf`, `numWords`, and `words`.

---

## InMemoryLookupCache

`InMemoryLookupCache` is the standard implementation of `VocabCache` and the one used by Word2Vec and ParagraphVectors by default. It stores everything in Java `HashMap` and `ArrayList` structures in the JVM heap.

You do not normally instantiate it yourself. After training, you obtain it from the model:

```java
Word2Vec model = new Word2Vec.Builder()
        .minWordFrequency(5)
        .layerSize(100)
        // ...
        .build();
model.fit();

VocabCache<VocabWord> vocabCache = model.getVocab();
```

---

## VocabWord

`VocabWord` is the element type stored in the cache. It carries:

| Field | Meaning |
|---|---|
| `word` | The string form of the token |
| `wordFrequency` | Total number of times this token appeared in the training corpus |
| `index` | The integer index used to look up this word's row in the embedding matrix |
| `docFrequency` | Number of documents (sentences) in which this token appeared |
| `huffmanPoint` | Encoding used by hierarchical softmax (if that training objective is used) |
| `huffmanCode` | Binary Huffman code for this word |

Access a `VocabWord` by string:

```java
VocabWord word = (VocabWord) vocabCache.wordFor("neural");
System.out.println("Frequency: " + word.getWordFrequency());
System.out.println("Index: " + word.getIndex());
```

---

## Inspecting the Vocabulary After Training

### Vocabulary Size

```java
int size = vocabCache.numWords();
System.out.println("Vocabulary size: " + size);
```

### All Vocabulary Words

```java
Collection<String> allWords = vocabCache.words();
for (String word : allWords) {
    System.out.println(word);
}
```

### Check if a Word Is in Vocabulary

```java
boolean present = vocabCache.containsWord("embedding");
```

### Word Frequency

```java
VocabWord word = (VocabWord) vocabCache.wordFor("learning");
double freq = word.getWordFrequency();
System.out.println("'learning' appeared " + freq + " times");
```

### Word Index

The index is the row number in the embedding weight matrix (`syn0`). You can use it to look up the raw vector:

```java
int idx = vocabCache.indexOf("neural");
INDArray weights = ((InMemoryLookupTable) model.lookupTable()).getSyn0();
INDArray wordVector = weights.getRow(idx);
```

### Total Token Occurrences

```java
long total = vocabCache.totalWordOccurrences();
System.out.println("Total word occurrences in corpus: " + total);
```

---

## How minWordFrequency Filters the Vocabulary

During Word2Vec training, the pipeline processes the corpus in two passes:

1. **First pass (vocabulary building):** Every token produced by the `TokenizerFactory` is added to the cache via `addToken`. Its frequency counter is incremented each time it appears.

2. **Frequency filtering:** After the first pass, tokens with `wordFrequency < minWordFrequency` are removed from the vocab (though they remain as tokens). These words will not get embedding vectors.

3. **Second pass (training):** Only vocab words are used for training. Tokens that were filtered out are effectively ignored during the skip-gram / CBOW updates.

This means that if you set `minWordFrequency(5)` and a word appears 4 times, it will not have a vector in the trained model. Calling `model.getWordVector("that-word")` will return zeros, and `model.wordsNearest("that-word", 10)` will return an empty collection or throw.

```java
// Verify a word made it into the vocabulary before querying
if (vocabCache.containsWord("myword")) {
    double[] vector = model.getWordVector("myword");
}
```

---

## TF-IDF and Document Frequency

`VocabCache` also tracks document-level statistics used by TF-IDF vectorization:

```java
// Number of documents in which a word appeared
int docFreq = vocabCache.docAppearedIn("learning");

// Total number of documents seen
double totalDocs = vocabCache.totalNumberOfDocs();

// Inverse document frequency
double idf = vocabCache.idf("learning");
```

These statistics are populated automatically when you use a `BagOfWordsVectorizer` or similar vectorizer. When using only Word2Vec or ParagraphVectors, `docAppearedIn` may not be populated depending on the training configuration.

---

## Manually Adding Tokens and Words

In custom NLP components, you may need to add tokens and vocab words to a cache manually:

```java
InMemoryLookupCache cache = new InMemoryLookupCache();

// Step 1: Add a token (tracks the string and its frequency)
VocabWord tokenEntry = new VocabWord(1.0, "myword");
cache.addToken(tokenEntry);

// Step 2: If the word has met the frequency threshold,
//         promote it to a vocab word
cache.addWordToIndex(cache.numWords(), "myword");
cache.putVocabWord("myword");

// The special UNK token is always added first with index 0
cache.addWordToIndex(0, Word2Vec.UNK);
cache.putVocabWord(Word2Vec.UNK);
```

`Word2Vec.UNK` is the unknown word token. DL4J reserves index 0 for it. Words not in the vocabulary are mapped to this token's vector (which is typically a zero vector or a learned fallback representation).

---

## Accessing the VocabCache from a Serialized Model

When you save a Word2Vec model with `WordVectorSerializer` and load it back, the vocab cache is preserved:

```java
Word2Vec loaded = WordVectorSerializer.readWord2VecModel("/models/word2vec.bin");
VocabCache<?> vocab = loaded.getVocab();

System.out.println("Loaded vocabulary size: " + vocab.numWords());
System.out.println("Contains 'neural': " + vocab.containsWord("neural"));
```

You can also load just the vocabulary and weights without the full model:

```java
Pair<InMemoryLookupTable, VocabCache> pair =
        WordVectorSerializer.loadTxt(new File("/models/word2vec.txt"));

VocabCache vocab = pair.getSecond();
InMemoryLookupTable table = pair.getFirst();

// Walk all words with their vectors
for (int i = 0; i < vocab.numWords(); i++) {
    String word = vocab.wordAtIndex(i);
    INDArray vector = table.getSyn0().getRow(i);
    // ...
}
```

---

## Further Reading

- [NLP Overview](overview.md) — how VocabCache fits into the full pipeline
- [Word2Vec](word2vec.md) — the primary consumer of VocabCache
- [Tokenization](tokenization.md) — how tokens are produced before being added to the cache
- [Doc2Vec](doc2vec.md) — ParagraphVectors extends the vocabulary with document label tokens
