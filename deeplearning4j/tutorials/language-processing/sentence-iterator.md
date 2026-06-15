---
title: "Sentence Iterators"
description: "Text input for NLP — SentenceIterator, BasicLineIterator, FileSentenceIterator, and custom iterators"
---

# Sentence Iterators

A `SentenceIterator` is the entry point for raw text in Deeplearning4j's NLP pipeline. It abstracts over any text source — a file, a directory, an in-memory collection, a database result set, or a network stream — and presents each chunk of text to the training algorithm as a plain `String`.

The name "sentence" does not mean that the text returned must be a grammatical sentence. It means a unit of context for the NLP algorithm. Depending on your task, one iteration might return:

- A single sentence from a novel
- A tweet (140–280 characters)
- A news article (several paragraphs)
- A product review
- A log line

The key property is that the algorithm treats each string returned by the iterator as an independent context. Co-occurrence statistics are not gathered across iterator boundaries, so sentence boundaries matter for Word2Vec quality.

---

## The SentenceIterator Interface

```java
public interface SentenceIterator {
    String nextSentence();
    boolean hasNext();
    void reset();
    void finish();
    SentencePreProcessor getPreProcessor();
    void setPreProcessor(SentencePreProcessor preProcessor);
}
```

`reset()` restarts the iteration from the beginning. This is called at the start of each training epoch. Any iterator you implement must support `reset()` correctly — an iterator that cannot reset will silently produce no training data after the first epoch.

`finish()` is called when the iterator will no longer be used; use it to close file handles or network connections.

`setPreProcessor` attaches a `SentencePreProcessor`, which transforms each raw string before it is returned to the caller. The preprocessor runs inside `nextSentence()`.

---

## SentencePreProcessor

`SentencePreProcessor` is a single-method functional interface:

```java
public interface SentencePreProcessor {
    String preProcess(String sentence);
}
```

The most common use is lowercase normalization:

```java
iterator.setPreProcessor(sentence -> sentence.toLowerCase());
```

Other common transformations:
- Strip HTML tags before passing to the iterator
- Normalize Unicode (e.g., NFD to NFC)
- Replace numbers with a placeholder token

The preprocessor fires once per sentence, before tokenization. Token-level normalization (stripping punctuation, stemming) belongs in the `TokenizerFactory` instead.

---

## Built-In Implementations

### BasicLineIterator

Reads a plain text file where each line is treated as one sentence. This is the simplest and most efficient iterator for corpora that are already one-sentence-per-line, such as Wikipedia sentence dumps or preprocessed news corpora.

```java
// From a classpath resource
String path = new ClassPathResource("corpus.txt").getFile().getAbsolutePath();
SentenceIterator iterator = new BasicLineIterator(path);

// Directly from a File
SentenceIterator iterator = new BasicLineIterator(new File("/data/corpus.txt"));
```

`BasicLineIterator` does not apply any preprocessing by default. Attach a preprocessor for normalization:

```java
SentenceIterator iterator = new BasicLineIterator("/data/corpus.txt");
iterator.setPreProcessor(s -> s.toLowerCase().trim());
```

### LineSentenceIterator

Similar to `BasicLineIterator` but always constructed from a `File` object. Functionally equivalent; choose whichever is more convenient given how you have the path.

```java
SentenceIterator iterator = new LineSentenceIterator(new File("/data/corpus.txt"));
iterator.setPreProcessor(sentence -> sentence.toLowerCase());
```

### CollectionSentenceIterator

Iterates over an in-memory `Collection<String>`. Use this when your text data is already loaded into a list or set — for example, strings fetched from a database, API responses, or a small test corpus.

```java
List<String> tweets = Arrays.asList(
        "Loving the new DL4J release",
        "Word2Vec training is fast with multiple threads",
        "Neural embeddings make NLP so much easier"
);

SentenceIterator iterator = new CollectionSentenceIterator(tweets);
```

`reset()` simply resets the internal index, so repeated epochs work correctly without reloading data.

### FileSentenceIterator

Walks a directory (or a single file) and returns each line of every file encountered. Useful when your corpus is split across many files rather than stored in a single file.

```java
// Directory: iterates all files in the directory, line by line
SentenceIterator iterator = new FileSentenceIterator(new File("/data/corpus/"));

// Single file: equivalent to BasicLineIterator
SentenceIterator iterator = new FileSentenceIterator(new File("/data/corpus/shard_001.txt"));
```

The order in which files are processed depends on the filesystem. If reproducibility matters, sort your files explicitly and use a `CollectionSentenceIterator` backed by your sorted list of lines.

### UimaSentenceIterator

The most powerful iterator, backed by Apache UIMA and the ClearTK / OpenNLP pipelines. Unlike the file-based iterators above, it performs linguistic sentence segmentation rather than splitting on newlines. This handles:

- Text where multiple sentences appear on one line
- Abbreviations (Mr., U.S., etc.) that contain periods but do not end sentences
- Documents where sentence boundaries have not been pre-annotated

```java
// Simple creation: point at a directory of plain text documents
SentenceIterator iterator = UimaSentenceIterator.create("/data/documents/");
```

For more control over the UIMA analysis pipeline:

```java
SentenceIterator iterator = new UimaSentenceIterator(
        "/data/documents/",
        AnalysisEngineFactory.createEngine(
                AnalysisEngineFactory.createEngineDescription(
                        TokenizerAnnotator.getDescription(),
                        SentenceAnnotator.getDescription()
                )
        )
);
```

`UimaSentenceIterator` requires the `deeplearning4j-nlp-uima` artifact:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-nlp-uima</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

Use UIMA when your corpus is raw prose text and you need linguistically correct sentence boundaries. For corpora that are already pre-segmented into one sentence per line, the UIMA overhead is unnecessary.

---

## Choosing an Iterator

| Situation | Recommended iterator |
|---|---|
| Single file, one sentence per line | `BasicLineIterator` |
| Large file, custom sentence format | `LineSentenceIterator` with preprocessor |
| Corpus split across many files, one sentence per line | `FileSentenceIterator` |
| Text already in a Java collection | `CollectionSentenceIterator` |
| Raw prose where sentence boundaries must be detected | `UimaSentenceIterator` |
| Custom source (database, API, stream) | Custom implementation (see below) |

---

## Creating a Custom SentenceIterator

Extend `BaseSentenceIterator` and implement `nextSentence()`, `hasNext()`, and `reset()`. `BaseSentenceIterator` handles the preprocessor plumbing for you, so your `nextSentence()` can return the raw string and the base class will apply any attached preprocessor automatically.

```java
public class DatabaseSentenceIterator extends BaseSentenceIterator {

    private final Connection connection;
    private ResultSet resultSet;
    private boolean hasMore = true;

    public DatabaseSentenceIterator(Connection connection, String query) throws SQLException {
        this.connection = connection;
        PreparedStatement stmt = connection.prepareStatement(query);
        this.resultSet = stmt.executeQuery();
        advance();
    }

    private void advance() throws SQLException {
        hasMore = resultSet.next();
    }

    @Override
    public String nextSentence() {
        try {
            String text = resultSet.getString("text_column");
            advance();
            return text;
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public boolean hasNext() {
        return hasMore;
    }

    @Override
    public void reset() {
        // Re-execute the query to restart
        try {
            PreparedStatement stmt = connection.prepareStatement(query);
            this.resultSet = stmt.executeQuery();
            advance();
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }
}
```

Usage:

```java
SentenceIterator iterator = new DatabaseSentenceIterator(conn, "SELECT text_column FROM documents");
iterator.setPreProcessor(s -> s.toLowerCase());
```

---

## Thread Safety

DL4J's Word2Vec and ParagraphVectors implementations are multi-threaded. The built-in iterators (except `UimaSentenceIterator`) use internal synchronization to make `nextSentence()` thread-safe. When implementing a custom iterator, synchronize access to any shared mutable state in `nextSentence()` and `hasNext()`.

---

## Further Reading

- [NLP Overview](overview.md) — how SentenceIterator fits into the full pipeline
- [Tokenization](tokenization.md) — what happens to each string after the iterator produces it
- [Word2Vec](word2vec.md) — using a SentenceIterator with Word2Vec
- [Doc2Vec](doc2vec.md) — label-aware iterators for document embeddings
