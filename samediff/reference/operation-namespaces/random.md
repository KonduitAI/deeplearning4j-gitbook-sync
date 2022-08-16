# Random

## bernoulli

```java
INDArray bernoulli(double p, DataType datatype, long[] shape)

SDVariable bernoulli(double p, DataType datatype, long[] shape)
SDVariable bernoulli(String name, double p, DataType datatype, long[] shape)
```

Generate a new random INDArray, where values are randomly sampled according to a Bernoulli distribution,

with the specified probability. Array values will have value 1 with probability P and value 0 with probability

1-P.

* **p** - Probability of value 1
* **datatype** - Data type of the output variable
* **shape** - Shape of the new random INDArray, as a 1D array (Size: AtLeast(min=0))

## binomial

```java
INDArray binomial(int nTrials, double p, DataType datatype, long[] shape)

SDVariable binomial(int nTrials, double p, DataType datatype, long[] shape)
SDVariable binomial(String name, int nTrials, double p, DataType datatype, long[] shape)
```

Generate a new random INDArray, where values are randomly sampled according to a Binomial distribution,

with the specified number of trials and probability.

* **nTrials** - Number of trials parameter for the binomial distribution
* **p** - Probability of success for each trial
* **datatype** - Data type of the output variable
* **shape** - Shape of the new random INDArray, as a 1D array (Size: AtLeast(min=0))

## exponential

```java
INDArray exponential(double lambda, DataType datatype, long[] shape)

SDVariable exponential(double lambda, DataType datatype, long[] shape)
SDVariable exponential(String name, double lambda, DataType datatype, long[] shape)
```

Generate a new random INDArray, where values are randomly sampled according to a exponential distribution:

P(x) = lambda _exp(-lambda_ x)

* **lambda** - lambda parameter
* **datatype** - Data type of the output variable
* **shape** - Shape of the new random INDArray, as a 1D array (Size: AtLeast(min=0))

## logNormal

```java
INDArray logNormal(double mean, double stddev, DataType datatype, long[] shape)

SDVariable logNormal(double mean, double stddev, DataType datatype, long[] shape)
SDVariable logNormal(String name, double mean, double stddev, DataType datatype, long[] shape)
```

Generate a new random INDArray, where values are randomly sampled according to a Log Normal distribution,

i.e., `log(x) ~ N(mean, stdev)`

* **mean** - Mean value for the random array
* **stddev** - Standard deviation for the random array
* **datatype** - Data type of the output variable
* **shape** - Shape of the new random INDArray, as a 1D array (Size: AtLeast(min=0))

## normal

```java
INDArray normal(double mean, double stddev, DataType datatype, long[] shape)

SDVariable normal(double mean, double stddev, DataType datatype, long[] shape)
SDVariable normal(String name, double mean, double stddev, DataType datatype, long[] shape)
```

Generate a new random INDArray, where values are randomly sampled according to a Gaussian (normal) distribution,

N(mean, stdev)

* **mean** - Mean value for the random array
* **stddev** - Standard deviation for the random array
* **datatype** - Data type of the output variable
* **shape** - Shape of the new random INDArray, as a 1D array (Size: AtLeast(min=0))

## normalTruncated

```java
INDArray normalTruncated(double mean, double stddev, DataType datatype, long[] shape)

SDVariable normalTruncated(double mean, double stddev, DataType datatype, long[] shape)
SDVariable normalTruncated(String name, double mean, double stddev, DataType datatype, long[] shape)
```

Generate a new random INDArray, where values are randomly sampled according to a Gaussian (normal) distribution,

N(mean, stdev). However, any values more than 1 standard deviation from the mean are dropped and re-sampled

* **mean** - Mean value for the random array
* **stddev** - Standard deviation for the random array
* **datatype** - Data type of the output variable
* **shape** - Shape of the new random INDArray, as a 1D array (Size: AtLeast(min=0))

## uniform

```java
INDArray uniform(double min, double max, DataType datatype, long[] shape)

SDVariable uniform(double min, double max, DataType datatype, long[] shape)
SDVariable uniform(String name, double min, double max, DataType datatype, long[] shape)
```

Generate a new random INDArray, where values are randomly sampled according to a uniform distribution,

U(min,max)

* **min** - Minimum value
* **max** - Maximum value.
* **datatype** - Data type of the output variable
* **shape** - Shape of the new random INDArray, as a 1D array (Size: AtLeast(min=0))
