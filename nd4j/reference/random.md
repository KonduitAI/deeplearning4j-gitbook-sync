---
title: "Random Number Generation"
description: "Generating random INDArrays, probability distributions, seeding, and reproducibility in ND4J"
---

# Random Number Generation

ND4J provides a flexible random number generation (RNG) system that covers everything from simple uniform arrays to parameterised probability distributions. The same API works on CPU and GPU: the backend automatically dispatches to the correct native RNG kernel.


## Basic Random Arrays

The two most common entry points are `Nd4j.rand` and `Nd4j.randn`. Both accept a varargs shape and return a `FLOAT` array by default.

```java
import org.nd4j.linalg.api.ndarray.INDArray;
import org.nd4j.linalg.factory.Nd4j;

// Uniform random values in [0, 1) — shape 3 x 4
INDArray u = Nd4j.rand(3, 4);

// Standard normal values N(0, 1) — shape 5 x 5
INDArray n = Nd4j.randn(5, 5);

// 1D arrays
INDArray u1d = Nd4j.rand(100);
INDArray n1d = Nd4j.randn(100);
```

`rand` draws from the continuous uniform distribution on the half-open interval [0, 1). `randn` draws from the standard normal distribution with mean 0 and standard deviation 1.


## Typed Random Arrays

Pass a `DataType` as the first argument to control the element type. This is preferred in production code because it makes the precision explicit and avoids relying on the global default.

```java
import org.nd4j.linalg.api.buffer.DataType;

// FLOAT (32-bit) uniform — most common choice
INDArray fRand  = Nd4j.rand(DataType.FLOAT, 3, 4);

// DOUBLE (64-bit) uniform — use when higher precision is needed
INDArray dRand  = Nd4j.rand(DataType.DOUBLE, 3, 4);

// FLOAT16 — reduces memory footprint on GPU
INDArray hRand  = Nd4j.rand(DataType.FLOAT16, 3, 4);

// Normal distribution equivalents
INDArray fRandn = Nd4j.randn(DataType.FLOAT, 3, 4);
INDArray dRandn = Nd4j.randn(DataType.DOUBLE, 3, 4);
```

You can also pass a `long[]` shape if you have it in a variable:

```java
long[] shape = {8, 128, 128};
INDArray volume = Nd4j.rand(DataType.FLOAT, shape);
```

Integer `DataType` values (`INT32`, `INT64`, etc.) are not supported by `rand`/`randn`; use a floating-point type and cast afterwards if you need integers.


## Seeded Random for Reproducibility

For reproducible results — unit tests, debugging, paper experiments — set a seed on the global RNG instance before generating arrays.

```java
// Fix the seed before generating any random arrays
Nd4j.getRandom().setSeed(42L);

INDArray a = Nd4j.rand(DataType.FLOAT, 3, 4);
INDArray b = Nd4j.randn(DataType.FLOAT, 3, 4);

// Resetting to the same seed produces identical sequences
Nd4j.getRandom().setSeed(42L);

INDArray a2 = Nd4j.rand(DataType.FLOAT, 3, 4);
INDArray b2 = Nd4j.randn(DataType.FLOAT, 3, 4);

// a2 equals a, b2 equals b — element by element
System.out.println(a.equalsWithEps(a2, 1e-6));  // true
```

`Nd4j.getRandom()` returns the `Random` instance associated with the calling thread (see [Thread Safety](#thread-safety) below). Calling `setSeed` on it affects only subsequent calls on that thread.

You can also set a seed at JVM startup via the system property:

```bash
java -Dorg.nd4j.rng.seed=42 -jar your-app.jar
```


## Distributions

For distributions beyond uniform and normal, use `Nd4j.getDistributions()` or construct a `Distribution` object directly and pass it to `Nd4j.getExecutioner().exec(...)`. Distribution classes live in `org.nd4j.linalg.api.rng.distribution` and its sub-packages.

### NormalDistribution

```java
import org.nd4j.linalg.api.rng.distribution.Distribution;
import org.nd4j.linalg.api.rng.distribution.impl.NormalDistribution;

// N(mean=2.0, std=0.5) — 1000 samples in a 1D array
Distribution normal = new NormalDistribution(2.0, 0.5);
INDArray samples = Nd4j.rand(normal, 1000);
```

### UniformDistribution

```java
import org.nd4j.linalg.api.rng.distribution.impl.UniformDistribution;

// Uniform on [low, high)
Distribution uniform = new UniformDistribution(-1.0, 1.0);
INDArray u = Nd4j.rand(uniform, 4, 4);
```

### BinomialDistribution

The binomial distribution models the number of successes in `n` independent Bernoulli trials each with success probability `p`.

```java
import org.nd4j.linalg.api.rng.distribution.impl.BinomialDistribution;

// 10 trials, p=0.3, fill a 100-element array
Distribution binom = new BinomialDistribution(10, 0.3);
INDArray counts = Nd4j.rand(binom, 100);
```

### LogNormalDistribution

Samples drawn from log-normal are always positive, making it useful for modelling quantities that cannot be negative.

```java
import org.nd4j.linalg.api.rng.distribution.impl.LogNormalDistribution;

// Log-normal with underlying normal N(mu=0, sigma=1)
Distribution logNormal = new LogNormalDistribution(0.0, 1.0);
INDArray ln = Nd4j.rand(logNormal, 500);
```

### nextDouble on an INDArray

`Nd4j.getRandom().nextDouble(INDArray, Distribution)` fills an existing array in-place with samples from the given distribution and returns it:

```java
INDArray target = Nd4j.zeros(DataType.DOUBLE, 5, 5);
Distribution dist = new NormalDistribution(0.0, 1.0);

// Fill target in-place and return it
INDArray filled = Nd4j.getRandom().nextDouble(target, dist);
// filled == target (same object)
```

This avoids a separate allocation when you already have a pre-allocated buffer.


## Random Ops via the Executioner

For performance-critical code or GPU-friendly patterns, use the custom random ops available through `Nd4j.getExecutioner()`. These ops write directly into a provided output array without going through a Java `Distribution` wrapper.

### BernoulliDistribution Op

Fills an array element-wise: each element is set to 1.0 with probability `p` and 0.0 with probability `1 - p`.

```java
import org.nd4j.linalg.api.ops.random.impl.BernoulliDistribution;

INDArray out = Nd4j.zeros(DataType.FLOAT, 4, 4);
// p = 0.7
Nd4j.getExecutioner().exec(new BernoulliDistribution(out, 0.7));
// each element is 0.0 or 1.0
```

### GaussianDistribution Op

Fills an array with samples from N(mean, std). This is equivalent to `Nd4j.randn` but lets you specify mean and standard deviation.

```java
import org.nd4j.linalg.api.ops.random.impl.GaussianDistribution;

INDArray out = Nd4j.zeros(DataType.FLOAT, 3, 3);
double mean = 5.0, std = 2.0;
Nd4j.getExecutioner().exec(new GaussianDistribution(out, mean, std));
```

### UniformDistribution Op

Fills an array with samples from a uniform distribution on [low, high).

```java
import org.nd4j.linalg.api.ops.random.impl.UniformDistribution;

INDArray out = Nd4j.zeros(DataType.FLOAT, 6, 6);
Nd4j.getExecutioner().exec(new UniformDistribution(out, -5.0, 5.0));
```

These ops respect the current thread's RNG seed, so setting `Nd4j.getRandom().setSeed(...)` before executing them yields reproducible results.


## SameDiff Random Operations

[SameDiff](../samediff) is ND4J's automatic-differentiation engine. Its random operations build computation graph nodes that are executed lazily. Use them when building models that require random noise as part of the forward pass (e.g., dropout, variational autoencoders).

```java
import org.nd4j.autodiff.samediff.SameDiff;
import org.nd4j.linalg.api.buffer.DataType;

SameDiff sd = SameDiff.create();

// Standard normal N(0, 1)
var normal = sd.random.normal("noise", 0.0, 1.0, DataType.FLOAT, 3, 4);

// Uniform in [low, high)
var uniform = sd.random.uniform("u", 0.0, 1.0, DataType.FLOAT, 3, 4);

// Bernoulli with success probability p
var bernoulli = sd.random.bernoulli("mask", 0.5, DataType.FLOAT, 3, 4);

// Truncated normal — normal values clipped to [-2*std, +2*std]
var truncNorm = sd.random.truncatedNormal("tn", 0.0, 1.0, DataType.FLOAT, 3, 4);
```

The shape can also be supplied as a `long[]`:

```java
long[] shape = {32, 128};
var z = sd.random.normal("z", 0.0, 1.0, DataType.FLOAT, shape);
```

To execute and retrieve the concrete value:

```java
sd.outputSingle(null, "noise");   // returns an INDArray
```

SameDiff random nodes participate fully in gradient computation. For example, a reparameterised sample `z = mean + std * epsilon` where `epsilon` is `sd.random.normal(...)` correctly propagates gradients through `mean` and `std`.

### Setting a SameDiff Seed

Set the seed on the underlying ND4J RNG before calling `outputSingle` (or any execution method):

```java
Nd4j.getRandom().setSeed(123L);
INDArray result = sd.outputSingle(null, "noise");
```


## Thread Safety

ND4J maintains one `Random` instance per thread. `Nd4j.getRandom()` returns the instance for the **calling thread**. This design means:

- Random generation is thread-safe by default: concurrent threads each draw from their own independent RNG state.
- Seeding is per-thread: calling `setSeed` on one thread does not affect any other thread's RNG.
- Parallel workloads (multi-threaded data loaders, parallel training) will each produce different sequences unless seeds are set independently on each thread.

To seed all threads identically at startup in a controlled test environment:

```java
// In a @BeforeAll or application init block, seed the main thread
Nd4j.getRandom().setSeed(0L);

// For worker threads created by an Executor, set the seed inside the task
ExecutorService pool = Executors.newFixedThreadPool(4);
for (int i = 0; i < 4; i++) {
    final int id = i;
    pool.submit(() -> {
        Nd4j.getRandom().setSeed(id);   // each thread gets a distinct seed
        // ... generate arrays ...
    });
}
```

Using distinct seeds per thread rather than the same seed prevents correlated streams across threads, which can introduce bias in stochastic training.


## Reproducibility

Getting fully reproducible results in deep learning involves several layers.

**1. Set the ND4J seed early.** Call `Nd4j.getRandom().setSeed(seed)` before any array creation, before building the model, and before the training loop.

**2. Set the DL4J seed.** When constructing a neural network, pass the same seed to the network configuration:

```java
MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
    .seed(42)
    // ... layer definitions ...
    .build();
```

This controls weight initialisation inside DL4J layers, which internally calls back into the ND4J RNG.

**3. Control framework-level parallelism.** By default, ND4J may use multiple CPU threads for parallelised BLAS operations. The order of floating-point reductions can vary across runs even with the same seed. To force a single thread during debugging:

```java
// Limit OpenMP threads (must be set before any computation)
System.setProperty("omp.num_threads", "1");
```

**4. GPU non-determinism.** CUDA operations such as atomics and certain convolution algorithms are non-deterministic at the hardware level. Enforcing full determinism on GPU is an active area of engineering. For experimentation, run on CPU to ensure bit-for-bit reproducibility.

**5. Isolate randomness in tests.** In JUnit tests, reset the seed in `@BeforeEach`:

```java
@BeforeEach
void resetSeed() {
    Nd4j.getRandom().setSeed(12345L);
}
```

This ensures each test case starts from the same RNG state regardless of test execution order.


## Quick Reference

### Array creation

| Method | Distribution | Notes |
|---|---|---|
| `Nd4j.rand(long...)` | Uniform [0, 1) | Default `FLOAT` type |
| `Nd4j.rand(DataType, long...)` | Uniform [0, 1) | Explicit type |
| `Nd4j.randn(long...)` | Normal N(0, 1) | Default `FLOAT` type |
| `Nd4j.randn(DataType, long...)` | Normal N(0, 1) | Explicit type |
| `Nd4j.rand(Distribution, long...)` | Any `Distribution` | Parameterised |

### Distribution classes (`org.nd4j.linalg.api.rng.distribution.impl`)

| Class | Parameters | Description |
|---|---|---|
| `NormalDistribution` | `mean, std` | Gaussian distribution |
| `UniformDistribution` | `low, high` | Continuous uniform |
| `BinomialDistribution` | `n, p` | Binomial trials |
| `LogNormalDistribution` | `mu, sigma` | Log-normal; always positive |

### Random ops (`org.nd4j.linalg.api.ops.random.impl`)

| Op class | Parameters | Description |
|---|---|---|
| `BernoulliDistribution` | `output, p` | Fill with 0/1 by probability |
| `GaussianDistribution` | `output, mean, std` | Fill with Gaussian samples |
| `UniformDistribution` | `output, low, high` | Fill with uniform samples |

### SameDiff random ops

| Method | Description |
|---|---|
| `sd.random.normal(name, mean, std, type, shape)` | Normal N(mean, std) |
| `sd.random.uniform(name, low, high, type, shape)` | Uniform [low, high) |
| `sd.random.bernoulli(name, p, type, shape)` | Bernoulli 0/1 |
| `sd.random.truncatedNormal(name, mean, std, type, shape)` | Truncated normal |

---

For weight initialisation strategies that use these random primitives under the hood, see the DL4J layer configuration page. For memory management implications of allocating large random arrays, see the [Memory and Workspaces](../core-concepts/memory-and-workspaces) page.
