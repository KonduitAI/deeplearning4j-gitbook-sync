---
description: Setting up clion for modifying the libnd4j code base
---

# How to Setup CLion

### Overview

In order to setup clion, we need to configure the cmake defaults to ensure that tests can be built. Normally this is setup by the [libnd4j build script](how-to-setup-clion.md#undefined). A general tutorial on how to configure cmake profiles  for clion can be found [here](https://www.jetbrains.com/clion/features/cmake-support.html). When configuring cmake to run, there are generally a few steps to follow.

This will in general include setting up the cmake gtest integration (we use google test for our test suite)

1. Setup a toolchain. This will depend on your OS. More can be found [here](https://www.jetbrains.com/help/clion/how-to-create-toolchain-in-clion.html). This will cover the compiler, debugger and other needed additional software to enable clion to manage your cmake project.
2. After configuration, let cmake build/index the files. It takes time to setup the files, auto complete and other expected functionality provided by the IDE. Ensure this is done by keeping an eye on the bottom right of the IDE to ensure all tasks are complete.
3. After your clion environment is setup, you may put the following cmake configuration for CPU:  -DSD\_CPU=true -DSD\_BUILD\_TESTS=true -DSD\_X86\_BUILD=true -DSD\_ALL\_OPS=true -DSD\_ARCH=x86-64 -DSD\_X86\_BUILD=true -DSD\_SHARED\_LIB=true
4. The above will configure your IDE to build a shared library for intel cpus as well as configure the gtest setup to run. Please ensure that you read about how to [configure cmake profiles](https://www.jetbrains.com/clion/features/cmake-support.html) (as linked above)
5. Now you should be able to make modifications backed by the IDE. Note that you can also run tests under[ tests_cpu/layers_tests](https://github.com/eclipse/deeplearning4j/tree/b838f7206e8ffc2addd795f09d85fb3815de7a37/libnd4j/tests\_cpu/layers\_tests)
6. In order to run all tests, you may run the [AllTests.cpp entry point](https://github.com/eclipse/deeplearning4j/blob/6dc7e2f08f1222d4384a5bc320f554fc3f3ef405/libnd4j/tests\_cpu/layers\_tests/AllTests.cpp). In order to run the tests (or even a specific test) just right click on any test and click the green arrow option that appears in the dropdown similar to "Run AllTests.." or something like that. For more information on this, please see [here](https://www.jetbrains.com/help/clion/creating-google-test-run-debug-configuration-for-test.html#add-google-tests).
7. When running tests, note that it may take a while. It will build a whole executable compiling the relevant parts of the code base (or if necessary, the whole code base) in order to run the tests.



