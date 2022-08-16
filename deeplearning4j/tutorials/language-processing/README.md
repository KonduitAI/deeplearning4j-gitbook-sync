---
description: Overview of language processing in DL4J
---

# Language Processing

Although not designed to be comparable to tools such as Stanford CoreNLP or NLTK, deepLearning4J does include some core text processing tools that are described here.

Deeplearning4j's NLP support contains interfaces for different NLP libraries. A user wraps third party libraries via our interfaces. Deeplearning4j as of M1, does not support any 3rd party libraries directly. This is due to the lack of maintenance and custom work needed to make this work well for users. Instead, we expose interfaces to allow users to implement their own tokenizers.

## SentenceIterator

There are several steps involved in processing natural language. The first is to iterate over your corpus to create a list of documents, which can be as short as a tweet, or as long as a newspaper article. This is performed by a SentenceIterator, which will appear like this:

```java
// Gets Path to Text file
String filePath = new File(dataLocalPath,"raw_sentences.txt").getAbsolutePath();
// Strip white space before and after for each line
SentenceIterator iter = new BasicLineIterator(filePath);
```

The SentenceIterator encapsulates a corpus or text, organizing it, say, as one Tweet per line. It is responsible for feeding text piece by piece into your natural language processor. The SentenceIterator is not analogous to a similarly named class, the DatasetIterator, which creates a dataset for training a neural net. Instead it creates a collection of strings by segmenting a corpus.

## Tokenizer

A Tokenizer further segments the text at the level of single words, also alternatively as n-grams. ClearTK contains the underlying tokenizers, such as parts of speech (PoS) and parse trees, which allow for both dependency and constituency parsing, like that employed by a recursive neural tensor network (RNTN).

A Tokenizer is created and wrapped by a [TokenizerFactory](https://github.com/eclipse/deeplearning4j/blob/6f027fd5075e3e76a38123ae5e28c00c17db4361/deeplearning4j-scaleout/deeplearning4j-nlp/src/main/java/org/deeplearning4j/text/tokenization/tokenizerfactory/UimaTokenizerFactory.java). The default tokens are words separated by spaces. The tokenization process also involves some machine learning to differentiate between ambibuous symbols like . which end sentences and also abbreviate words such as Mr. and vs.

Both Tokenizers and SentenceIterators work with Preprocessors to deal with anomalies in messy text like Unicode, and to render such text, say, as lowercase characters uniformly.

```java
 public static void main(String[] args) throws Exception {

        dataLocalPath = DownloaderUtility.NLPDATA.Download();
        // Gets Path to Text file
        String filePath = new File(dataLocalPath,"raw_sentences.txt").getAbsolutePath();

        log.info("Load & Vectorize Sentences....");
        // Strip white space before and after for each line
        SentenceIterator iter = new BasicLineIterator(filePath);
        // Split on white spaces in the line to get words
        TokenizerFactory t = new DefaultTokenizerFactory();

        /*
            CommonPreprocessor will apply the following regex to each token: [\d\.:,"'\(\)\[\]|/?!;]+
            So, effectively all numbers, punctuation symbols and some special symbols are stripped off.
            Additionally it forces lower case for all tokens.
         */
        t.setTokenPreProcessor(new CommonPreprocessor());
```

## Vocab

Each document has to be tokenized to create a vocab, the set of words that matter for that document or corpus. Those words are stored in the vocab cache, which contains statistics about a subset of words counted in the document, the words that "matter". The line separating significant and insignifant words is mobile, but the basic idea of distinguishing between the two groups is that words occurring only once (or less than, say, five times) are hard to learn and their presence represents unhelpful noise.

The vocab cache stores metadata for methods such as Word2vec and Bag of Words, which treat words in radically different ways. Word2vec creates representations of words, or neural word embeddings, in the form of vectors that are hundreds of coefficients long. Those coefficients help neural nets predict the likelihood of a word appearing in any given context; for example, after another word. Here's Word2vec, configured:

```java
package org.deeplearning4j.examples.nlp.word2vec;

import org.deeplearning4j.examples.download.DownloaderUtility;
import org.deeplearning4j.models.word2vec.Word2Vec;
import org.deeplearning4j.text.sentenceiterator.BasicLineIterator;
import org.deeplearning4j.text.sentenceiterator.SentenceIterator;
import org.deeplearning4j.text.tokenization.tokenizer.preprocessor.CommonPreprocessor;
import org.deeplearning4j.text.tokenization.tokenizerfactory.DefaultTokenizerFactory;
import org.deeplearning4j.text.tokenization.tokenizerfactory.TokenizerFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.util.Collection;

/**
 * Created by agibsonccc on 10/9/14.
 *
 * Neural net that processes text into wordvectors. See below url for an in-depth explanation.
 * https://deeplearning4j.org/word2vec.html
 */
public class Word2VecRawTextExample {

    private static Logger log = LoggerFactory.getLogger(Word2VecRawTextExample.class);

    public static String dataLocalPath;


    public static void main(String[] args) throws Exception {

        dataLocalPath = DownloaderUtility.NLPDATA.Download();
        // Gets Path to Text file
        String filePath = new File(dataLocalPath,"raw_sentences.txt").getAbsolutePath();

        log.info("Load & Vectorize Sentences....");
        // Strip white space before and after for each line
        SentenceIterator iter = new BasicLineIterator(filePath);
        // Split on white spaces in the line to get words
        TokenizerFactory t = new DefaultTokenizerFactory();

        /*
            CommonPreprocessor will apply the following regex to each token: [\d\.:,"'\(\)\[\]|/?!;]+
            So, effectively all numbers, punctuation symbols and some special symbols are stripped off.
            Additionally it forces lower case for all tokens.
         */
        t.setTokenPreProcessor(new CommonPreprocessor());

        log.info("Building model....");
        Word2Vec vec = new Word2Vec.Builder()
                .minWordFrequency(5)
                .iterations(1)
                .layerSize(100)
                .seed(42)
                .windowSize(5)
                .iterate(iter)
                .tokenizerFactory(t)
                .build();

        log.info("Fitting Word2Vec model....");
        vec.fit();

        log.info("Writing word vectors to text file....");

        // Prints out the closest 10 words to "day". An example on what to do with these Word Vectors.
        log.info("Closest Words:");
        Collection<String> lst = vec.wordsNearestSum("day", 10);
        log.info("10 Words closest to 'day': {}", lst);
    }
}
```

Once you obtain word vectors, you can feed them into a deep net for classification, prediction, sentiment analysis and the like.
