---
title: Platform Summary
short_title: Platform Summary
description: Summary of deeplearning4j projects on different platforms
category: Multi-Project
weight: 10
---

# [Overview](#overview)

This page provides a summary of where to get started with deeplearning4j on different platforms.
It will also make developers aware of what to expect when running ML workloads on different platforms.


# [Windows](#windows)

The deeplearning4j suite of projects supports running on intel cpus and cuda for NVIDIA GPUs.
For intel cpus, we provide avx2 and avx512 binaries. For more on these optimizations please see
[here](config/backends/cpu.md)

Our cuda support is described in our [cuda](#cuda) section. If you would like to use cudnn, please see
our [section](../config/backends/config-cudnn) on this as well.



# [Mac OSX](#macosx)

For mac osx support, we only currently support intel cpus.
For configuration of running on cpus please see [here](../config/backends/cpu.md)
Support for the M1 set of devices will come at a later date.
Nothing blocks this in theory, mostly just time. Recent reports suggest
java support is still very new on the mac osx m1. There is an issue [here](https://github.com/eclipse/deeplearning4j/issues/9258)
tracking support for the M1.




# [Linux](#linux)

On linux, the project supports a wide variety of configurations including:
1. armhf, armv8/arm64 (including raspberry pi) - note on ARM devices, [armcompute optimizations are supported as well](https://github.com/ARM-software/ComputeLibrary)
2. intel cpus with avx2/512 optimizations - read more [here](../config/backends/cpu.md)
3. cuda 11.0 and 11.2



Note for jetson nano users: due to nvidia not supporting cuda 11 we are waiting to see if we can avoid
backporting anything to cuda 10. Please see [this topic](https://community.konduit.ai/t/cuda-on-jetson-nano/1364)
for more information.

# [Android](#android)

On android we support the following architectures:
1. armhf/armv8: as with linux [armcompute optimizations are supported as well](https://github.com/ARM-software/ComputeLibrary)
2. x86_64:  we don't support any particular optimizations for android x86_64 at this moment. If you are interested in that,
please [file an issue here](https://github.com/eclipse/deeplearning4j/issues/new)

On android, we've built native support using the android r21d ndk.

Note for newer users: our current libnd4j binaries are around 150MB in size. We unfortunately need to address this.
Please feel free to comment on [our support forums](https://community.konduit.ai/) or [this github issue](https://github.com/eclipse/deeplearning4j/issues/8912)



# [CUDA](#cuda)

Deeplearning4j (as of 1.0.0-M1) provides support for cuda 11.0 and 11.2.
Anything more than that requires compiling deeplearning4j (including the c++ library libnd4j)
from source. If you need support for other platforms, please see our [building from source guide](../getting-started/build-from-source)

# AMD ROCOM:

We do not support this at this time. If you are interested in helping us add support for this,
please feel free to discuss it on our [forums](https://community.konduit.ai/)
