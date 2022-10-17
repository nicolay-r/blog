---
layout: post
title: "Setup JAX framework with GPU support"
description: "Setup JAX framework with GPU support"
category: POST
tags: [JAX, T5x, jaxlib, jax, google, cudnn, nvcc, cuda, gpu, autograd, xla, flaxformer, google-research, nvidia]
---

Calculation of the derivatives plays a significant role in neural networks tuning. 
The computational effectivenes is very crusial considering a large models and ability to train them.
Besides the `CPU` and one of the most common option for calculations, recent advances finds a significant application of `GPU` and `TPUs`
mostly because of a potentially greater performance vs. the central processing unit.
Obvioulsy, such features would be available without a special software support, networks compilation towards the targeted device and platform.

<!--more-->

`JAX`, rougly speaking, is a library, proposed and maintained by Google which combines 
[autograd](https://github.com/hips/autograd)
and [XLA](https://www.tensorflow.org/xla) 
compiler in order to bring neural networks  onto the computational devices in a most efficient way.
However, speaking about `GPU` and more nn oriented devices as `TPU`, it is pretty important to perform a proper library setup in order to 
make them available for `JAX`.
`JAX` is widely used for transformers, which significant amount of applications could be found for 
[T5](https://github.com/google-research/t5x), with their variations and optimisations formed into another 
[flaxformer](https://github.com/google/flaxformer) project.

In this post we address on the issue you may encountered with once decided to apply this library for `GPU` devices.
The latter is required to make library frienly and familiar with such frameworks as: NVidia CUDA compiler (`ncdu`), CUDA DNN (`cudnn`).
We took the steps and refer to the experience proposed in [JAX installation with Nvidia CUDA and cudNN support](https://www.youtube.com/watch?v=auksaSl8jlM) video.

Let's get started!

The problem that you may enconter first is that you got installed the ordinary version of the related library dubbed as `jaxlib`
At first, let's say we have installed:
```
jax==0.3.21
jaxlib==0.3.15
```
Here is list of actions required to be performed in order to make it familiar with `GPU` devices:

1. Uninstall `jax` and `jaxlib`:
```
pip uninstall jax jaxlib
```
2. `cudnn` version checkout:
```
cat /usr/include/x86_64-linux-gnu/cudnn_v*.h | grep CUDNN
```
3. Clean install from [list of laxlib-cudnn-cuda](https://storage.googleapis.com/jax-releases/jax_cuda_releases.html):
```
pip install --upgrade jax==0.3.15 jaxlib==0.3.15+cuda11.cudnn82 -f https://storage.googleapis.com/jax-releases/cuda11/jaxlib-0.3.15+cuda11.cudnn82-cp39-none-manylinux2014_x86_64.whl
```
4. check out:
```
>>> import jax
>>> jax.devices()
[GpuDevice(id=0, process_index=0), GpuDevice(id=1, process_index=0), GpuDevice(id=2, process_index=0), GpuDevice(id=3, process_index=0)]
```

# Reference

[JAX installation with Nvidia CUDA and cudNN support](https://www.youtube.com/watch?v=auksaSl8jlM)
