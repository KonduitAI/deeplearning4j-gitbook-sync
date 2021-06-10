---
title: Tokenization
short_title: Tokenization
description: Breaking text into individual words for language processing in DL4J.
category: Language Processing
weight: 10
---

# Tokenization

Notes to write on: 1. Tokenizer factory interface 2. Tokenizer interface 2. How to write your own factory and tokenizer

## Tokenization

### What is Tokenization?

Tokenization is the process of breaking text down into individual words. Word windows are also composed of tokens. [Word2Vec](word2vec) can output text windows that comprise training examples for input into neural nets, as seen here.

### Example

Here's an example of tokenization done with DL4J tools:

```java
TokenizerFactory tokenizerFactory = new DefaultTokenizerFactory();
Tokenizer tokenizer = tokenizerFactory.tokenize("mystring");

//iterate over the tokens
while(tokenizer.hasMoreTokens()) {
      String token = tokenizer.nextToken();
}

//get the whole list of tokens
List<String> tokens = tokenizer.getTokens();
```

The above snippet creates a tokenizer capable of stemming.

In Word2Vec, that's the recommended a way of creating a vocabulary, because it averts various vocabulary quirks, such as the singular and plural of the same noun being counted as two different words.

