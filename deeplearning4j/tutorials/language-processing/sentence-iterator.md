---
description: Iteration of words, documents, and sentences for language processing in DL4J.
---

# Sentence Iterator

A sentence iterator is used in both [Word2vec](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/tutorials/language-processing/word2vec.md) and [Bag of Words](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/tutorials/language-processing/deeplearning4j-nlp/bagofwords-tf-idf.html).

It feeds bits of text into a neural network in the form of vectors, and also covers the concept of documents in text processing.

In natural-language processing, a document or sentence is typically used to encapsulate a context which an algorithm should learn.

A few examples include analyzing Tweets and full-blown news articles. The purpose of the [sentence iterator](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/tutorials/language-processing/deeplearning4j-nlp/doc/org/deeplearning4j/word2vec/sentenceiterator/SentenceIterator.html) is to divide text into processable bits. Note the sentence iterator is input agnostic. So bits of text (a document) can come from a file system, the Twitter API or Hadoop.

Depending on how input is processed, the output of a sentence iterator will then be passed to a [tokenizer](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/tutorials/language-processing/tokenization.md) for the processing of individual tokens, which are usually words, but could also be ngrams, skipgrams or other units. The tokenizer is created on a per-sentence basis by a [tokenizer factory](https://app.gitbook.com/s/-LsGrpMiOeoMSFYK0VJQ-714541269/deeplearning4j/tutorials/language-processing/tokenization.md#example). The tokenizer factory is what is passed into a text-processing vectorizer.

Some typical examples are below:

```java
SentenceIterator iter = new LineSentenceIterator(new File("your file"));
```

This assumes that each line in a file is a sentence.

You can also do list of strings as sentence as follows:

```java
Collection<String> sentences = ...;
SentenceIterator iter = new CollectionSentenceIterator(sentences);
```

This will assume that each string is a sentence (document). Remember this could be a list of Tweets or articles -- both are applicable.

You can iterate over files as follows:

```java
SentenceIterator iter = new FileSentenceIterator(new File("your dir or file"));
```

This will parse the files line by line and return individual sentences on each one.

For anything complex, we recommend any pipeline that can implement more in depth support than space separated tokens.
