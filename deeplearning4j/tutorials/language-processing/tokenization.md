---
title: "Tokenization"
description: "Tokenizer factories in Deeplearning4j — DefaultTokenizerFactory, custom tokenizers, and preprocessors"
---

# Tokenization

Tokenization is the process of splitting a sentence string into a sequence of individual units — tokens — that the NLP algorithm processes. In Word2Vec and ParagraphVectors, tokens are almost always individual words. The tokenizer also provides the hook for token-level normalization: stripping punctuation, lowercasing, and filtering stop words happen inside the tokenizer layer.

---

## The Two-Level Interface

DL4J tokenization is organized around two interfaces that work together:

**`TokenizerFactory`** is a stateless factory. It holds configuration (such as the attached preprocessor) and produces a new `Tokenizer` instance for each sentence:

```java
public interface TokenizerFactory {
    Tokenizer create(String toTokenize);
    Tokenizer create(InputStream toTokenize);
    void setTokenPreProcessor(TokenPreProcess preProcessor);
    TokenPreProcess getTokenPreProcessor();
}
```

**`Tokenizer`** processes one sentence. It is created fresh for each sentence by the factory:

```java
public interface Tokenizer {
    boolean hasMoreTokens();
    int countTokens();
    String nextToken();
    List<String> getTokens();
    void setTokenPreProcessor(TokenPreProcess tokenPreProcessor);
}
```

You supply a `TokenizerFactory` to the model builder. The model calls `factory.create(sentence)` for each sentence the iterator produces, then calls `getTokens()` on the resulting tokenizer to extract the token list. You never call tokenizer methods directly when training.

---

## DefaultTokenizerFactory

The standard tokenizer for most Word2Vec and ParagraphVectors workloads. It splits on whitespace (and optionally punctuation) using a simple regex-based tokenizer internally.

```java
TokenizerFactory tokenizerFactory = new DefaultTokenizerFactory();
```

By default it splits on whitespace only. Attach a preprocessor to also handle punctuation and case:

```java
TokenizerFactory tokenizerFactory = new DefaultTokenizerFactory();
tokenizerFactory.setTokenPreProcessor(new CommonPreprocessor());
```

This is the recommended starting configuration for English text corpora.

---

## Token Preprocessors

A `TokenPreProcess` applies a transformation to each token string before it is returned by the tokenizer. The interface has one method:

```java
public interface TokenPreProcess {
    String preProcess(String token);
}
```

DL4J ships three built-in preprocessors:

### CommonPreprocessor

The most widely used preprocessor. It:
1. Lowercases the entire token
2. Strips all characters that are not letters or digits (removes punctuation, special characters)

```java
tokenizerFactory.setTokenPreProcessor(new CommonPreprocessor());
```

Use `CommonPreprocessor` as your default. It removes punctuation that would otherwise create spurious vocabulary entries like `"word."` and `"word,"` distinct from `"word"`.

### LowCasePreProcessor

Lowercases the token without removing any characters. Use this when punctuation is semantically meaningful in your domain (e.g., code tokens, chemical names) but you still want case normalization.

```java
tokenizerFactory.setTokenPreProcessor(new LowCasePreProcessor());
```

### EndingPreProcessor

Strips common English word endings (suffixes) using a simple rule-based approach. This is a lightweight alternative to full stemming: it catches common inflections (`-ing`, `-ly`, `-ed`, `-s`) without the overhead of a stemmer.

```java
tokenizerFactory.setTokenPreProcessor(new EndingPreProcessor());
```

Note that `EndingPreProcessor` does not lowercase. Chain it with `LowCasePreProcessor` if you need both:

```java
// Chain two preprocessors manually
TokenPreProcess lowerCase = new LowCasePreProcessor();
TokenPreProcess endings = new EndingPreProcessor();

tokenizerFactory.setTokenPreProcessor(token -> endings.preProcess(lowerCase.preProcess(token)));
```

---

## NGramTokenizerFactory

Wraps another `TokenizerFactory` and produces n-gram tokens in addition to unigrams. An n-gram is a contiguous sequence of N tokens. Adding bigrams (2-grams) to a Word2Vec vocabulary allows the model to learn representations for common multi-word expressions like "New York" or "machine learning" as single units.

```java
TokenizerFactory base = new DefaultTokenizerFactory();
base.setTokenPreProcessor(new CommonPreprocessor());

// Produce unigrams and bigrams (minN=1, maxN=2)
TokenizerFactory ngrams = new NGramTokenizerFactory(base, 1, 2);
```

For a sentence `"deep learning is powerful"`, this produces tokens:
`["deep", "learning", "is", "powerful", "deep learning", "learning is", "is powerful"]`

N-gram models significantly increase vocabulary size. Set `minWordFrequency` high enough to filter n-grams that appear only rarely:

```java
Word2Vec model = new Word2Vec.Builder()
        .minWordFrequency(10)  // Higher threshold for n-gram vocabularies
        .tokenizerFactory(ngrams)
        // ...
        .build();
```

---

## UimaTokenizerFactory

Uses Apache UIMA and OpenNLP under the hood to tokenize with full linguistic awareness — correct handling of abbreviations, contractions, hyphenated words, and other edge cases that defeat whitespace splitting.

```java
TokenizerFactory tokenizerFactory = new UimaTokenizerFactory();
```

`UimaTokenizerFactory` also supports stemming (reducing words to their root form):

```java
// tokenize("running") -> ["run"]
// tokenize("better")  -> ["good"]  (irregular)
TokenizerFactory tokenizerFactory = new UimaTokenizerFactory(true); // stem=true
```

Requires the UIMA artifact:

```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-nlp-uima</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```

The UIMA tokenizer produces higher-quality tokens but is significantly slower than `DefaultTokenizerFactory`. Benchmark both on your corpus before committing to UIMA for a large training run.

---

## Using a TokenizerFactory Directly

Outside of model training, you can use a `TokenizerFactory` to inspect how a sentence will be tokenized:

```java
TokenizerFactory tf = new DefaultTokenizerFactory();
tf.setTokenPreProcessor(new CommonPreprocessor());

Tokenizer tokenizer = tf.create("Hello, World! This is a test.");
List<String> tokens = tokenizer.getTokens();
System.out.println(tokens);
// [hello, world, this, is, a, test]
```

This is useful for debugging vocabulary issues: if words you expect are not appearing in your trained model, inspect the tokenizer output first to verify that your text is being processed as expected.

---

## Creating a Custom TokenizerFactory

Implement `TokenizerFactory` by extending `AbstractTokenizerFactory` and supplying a `Tokenizer` implementation:

```java
public class WhitespaceLowercaseTokenizerFactory extends AbstractTokenizerFactory {

    @Override
    public Tokenizer create(String toTokenize) {
        // Apply any attached preprocessor to the whole string first if needed
        return new WhitespaceLowercaseTokenizer(toTokenize, getTokenPreProcessor());
    }

    @Override
    public Tokenizer create(InputStream toTokenize) {
        try {
            String text = IOUtils.toString(toTokenize, StandardCharsets.UTF_8);
            return create(text);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}

public class WhitespaceLowercaseTokenizer extends AbstractTokenizer {

    private final List<String> tokens;
    private int index = 0;

    public WhitespaceLowercaseTokenizer(String sentence, TokenPreProcess preprocessor) {
        String[] parts = sentence.trim().split("\\s+");
        this.tokens = new ArrayList<>();
        for (String part : parts) {
            String processed = preprocessor != null ? preprocessor.preProcess(part) : part;
            if (processed != null && !processed.isEmpty()) {
                tokens.add(processed);
            }
        }
    }

    @Override
    public boolean hasMoreTokens() {
        return index < tokens.size();
    }

    @Override
    public String nextToken() {
        return tokens.get(index++);
    }

    @Override
    public List<String> getTokens() {
        return Collections.unmodifiableList(tokens);
    }

    @Override
    public int countTokens() {
        return tokens.size();
    }
}
```

---

## Stop Word Filtering

DL4J does not ship a built-in stop word list, but you can filter stop words in a custom `TokenPreProcess`:

```java
Set<String> stopWords = new HashSet<>(Arrays.asList(
        "the", "a", "an", "is", "it", "of", "in", "to", "and", "or"
));

tokenizerFactory.setTokenPreProcessor(token -> {
    String lower = token.toLowerCase().replaceAll("[^a-z0-9]", "");
    return stopWords.contains(lower) ? null : lower;
});
```

When the preprocessor returns `null`, the token is dropped. This behavior is implemented in the `AbstractTokenizer` base class — null tokens are not added to the token list.

Alternatively, `Word2Vec.Builder` exposes `.stopWords(List<String>)` which configures stop word exclusion directly on the model, applying the filter during vocabulary construction rather than tokenization:

```java
Word2Vec model = new Word2Vec.Builder()
        .stopWords(Arrays.asList("the", "a", "an"))
        // ...
        .build();
```

---

## Further Reading

- [NLP Overview](overview.md) — how tokenization fits into the full pipeline
- [Sentence Iterators](sentence-iterator.md) — the upstream stage that produces sentence strings
- [Vocabulary Cache](vocabulary-cache.md) — how tokens are counted and filtered into vocabulary
- [Word2Vec](word2vec.md) — using a TokenizerFactory for word embedding training
