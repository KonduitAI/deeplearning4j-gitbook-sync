---
title: "Contributing"
description: "How to contribute to Deeplearning4j — development setup, code style, pull requests, and community guidelines"
---

Contributions to Eclipse Deeplearning4j are welcome from everyone. This guide covers how to set up a development environment, follow project conventions, and submit changes.

## Repository Structure

All DL4J libraries live in the monorepo at [github.com/eclipse/deeplearning4j](https://github.com/eclipse/deeplearning4j):

- **deeplearning4j/** — neural network layers, training loop, evaluation
- **nd4j/** — N-dimensional array math backend (CPU and GPU)
- **libnd4j/** — native C++ compute engine compiled by CMake
- **datavec/** — data pipeline, ETL, and record readers
- **arbiter/** — hyperparameter optimization

A companion examples repository lives at [github.com/eclipse/deeplearning4j-examples](https://github.com/eclipse/deeplearning4j-examples).

## Ways to Contribute

- **Bug fixes** — look for issues labeled `bug` on the [issue tracker](https://github.com/eclipse/deeplearning4j/issues)
- **New features** — new layer types, training algorithms, optimizers, or backends
- **Documentation** — improve Javadoc, fix typos in docs, add examples
- **Performance** — profiling, benchmarking, identifying bottlenecks
- **Tests** — additional unit tests, edge cases, numerical gradient checks
- **Examples** — demonstrate a network architecture or application not yet covered

## Getting Started

### Fork and Clone

1. Fork the repository on GitHub (click **Fork** on the main repo page).
2. Clone your fork locally:

```bash
git clone https://github.com/YOUR_USERNAME/deeplearning4j.git
cd deeplearning4j
```

3. Add the upstream remote so you can pull in future changes:

```bash
git remote add upstream https://github.com/eclipse/deeplearning4j.git
```

### Build the Project

Follow the [Build from Source](./build-from-source) guide for full prerequisites and build steps. The short version for a CPU build:

```bash
# Build the native C++ backend
cd libnd4j
./buildnativeoperations.sh
export LIBND4J_HOME=$(pwd)
cd ..

# Build all Java modules
mvn clean install -DskipTests -Dmaven.javadoc.skip=true
```

A first build takes 15–45 minutes. Subsequent incremental builds are much faster.

### Required Tooling

- **JDK 11+** (JDK 17 recommended)
- **Maven 3.6.3+**
- **CMake 3.9+** and **gcc/g++ 7+**
- **Project Lombok plugin** for your IDE — without it the IDE shows false compilation errors everywhere
- **IntelliJ IDEA** (recommended) or Eclipse/NetBeans

## Development Workflow

### Create a Feature Branch

Never commit directly to `master`. Always branch from an up-to-date `master`:

```bash
git fetch upstream
git checkout -b my-feature upstream/master
```

Use a descriptive branch name, for example `fix/lstm-gradient-clip` or `feature/add-swiglu-activation`.

### Keep Your Branch Current

Rebase onto `upstream/master` regularly to avoid large merge conflicts:

```bash
git fetch upstream
git rebase upstream/master
```

### Commit Messages

Write commit messages in the imperative mood with a short subject line (under 72 characters). Add a body paragraph if the change needs explanation:

```
Add SwiGLU activation function

SwiGLU is used in large language model FFN blocks. This implements
the variant from Noam Shazeer's 2020 paper. Includes numerical
gradient check in ActivationsSmokeTest.
```

## Code Style

### Java

- Target Java 11 source compatibility (Java 17 features are not yet permitted in core modules).
- Follow existing code formatting — the project uses a 4-space indent, no tabs.
- Add **Javadoc** to all public methods and classes. Minimum: one-sentence description and `@param` / `@return` tags.
- Use **Lombok** annotations (`@Data`, `@Builder`, `@Slf4j`, etc.) consistently with surrounding code — do not mix manual boilerplate with Lombok in the same class.
- Avoid wildcard imports (`import org.nd4j.*`).
- If you add a new method or class: add an `@since` tag with the version (e.g., `@since 2.1.0`).
- Significant new functionality may include an `@author` tag, but this is optional.

### C++ (libnd4j)

- Follow the existing style in the surrounding file.
- Prefer RAII and smart pointers over raw `new`/`delete`.
- Document any non-obvious math or algorithm with an inline comment or link to a paper.

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Classes | UpperCamelCase | `MultiLayerNetwork` |
| Methods | lowerCamelCase | `computeGradientAndScore` |
| Constants | UPPER_SNAKE_CASE | `DEFAULT_LEARNING_RATE` |
| Packages | lowercase | `org.deeplearning4j.nn.layers` |

## Writing Tests

All non-trivial changes must include tests. DL4J uses JUnit 5.

### Unit Tests

Place tests in the same Maven module as the code under test, in `src/test/java/` mirroring the package structure:

```java
@Tag(TagNames.DL4J_OLD_API)
class MyNewLayerTest {

    @Test
    void forwardPassProducesExpectedShape() {
        // ...
    }
}
```

### Numerical Gradient Checks

Any new layer or loss function must pass a numerical gradient check. Use `GradientCheckUtil`:

```java
boolean passed = GradientCheckUtil.checkGradients(new GradientCheckUtil.MLNConfig()
    .net(net)
    .input(input)
    .labels(labels));
assertTrue(passed, "Gradient check failed");
```

Gradient checks confirm that analytic (backprop) gradients match finite-difference numerical gradients. A failing gradient check indicates a bug in the backward pass.

### Running Tests

Tests require the `dl4j-test-resources` repository (see [Build from Source](./build-from-source)):

```bash
# All tests, native CPU backend
mvn clean test -P testresources,test-nd4j-native

# Tests for a single module
mvn clean test -P testresources,test-nd4j-native \
    -pl deeplearning4j/deeplearning4j-core
```

## Creating a Pull Request

1. Push your branch to your fork:

```bash
git push origin my-feature
```

2. Open a pull request from your branch to `eclipse/deeplearning4j:master` on GitHub.

3. Fill out the PR description with:
   - **What** was changed and **why**
   - How to test or reproduce the fix
   - Any relevant issue numbers (`Fixes #1234`)

4. Ensure CI passes. The test suite runs automatically on each push. Do not merge until all required checks are green.

5. Address reviewer feedback by pushing additional commits to the same branch — do not force-push a branch that is under review.

6. A maintainer will merge the PR once it is approved.

### PR Checklist

- [ ] Branch is based on current `upstream/master`
- [ ] Code compiles with `mvn clean install -DskipTests`
- [ ] New or changed behavior is covered by tests
- [ ] New public API has Javadoc
- [ ] Numerical gradient checks pass for any new layer or loss
- [ ] Eclipse CLA is signed (first-time contributors only — see below)

## Reporting Issues

File bugs and feature requests at [github.com/eclipse/deeplearning4j/issues](https://github.com/eclipse/deeplearning4j/issues).

A useful bug report includes:

- DL4J version (or commit hash if built from source)
- Java version and OS
- A minimal reproducible example — the shorter the better
- Full stack trace if an exception is thrown
- Expected vs. actual behavior

For examples bugs, use [github.com/eclipse/deeplearning4j-examples/issues](https://github.com/eclipse/deeplearning4j-examples/issues) instead.

## Eclipse Foundation CLA

Eclipse Deeplearning4j is an Eclipse Foundation project. Before your first pull request can be merged, you must sign the **Eclipse Contributor Agreement (ECA)**:

1. Create an account at [accounts.eclipse.org](https://accounts.eclipse.org).
2. Sign the ECA at [eclipse.org/legal/ECA.php](https://www.eclipse.org/legal/ECA.php).
3. Ensure the email address on your GitHub account matches the email registered with the Eclipse Foundation.

The CLA check is automated — the Eclipse bot will comment on your PR if the ECA is missing.

## Community Channels

| Channel | Purpose |
|---------|---------|
| [GitHub Issues](https://github.com/eclipse/deeplearning4j/issues) | Bug reports, feature requests |
| [GitHub Discussions](https://github.com/eclipse/deeplearning4j/discussions) | Questions, design discussions |
| [Gitter](https://gitter.im/deeplearning4j/deeplearning4j) | Real-time chat with maintainers and community |
| [Early Adopters Gitter](https://gitter.im/deeplearning4j/deeplearning4j/earlyadopters) | Build issues, bleeding-edge questions |

When asking for help, include your DL4J version, OS, and a minimal reproducible example. The more context you provide, the faster someone can assist you.
