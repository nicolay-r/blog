---
layout: post
title: "Setup JAX framework with GPU support"
description: "Setup JAX framework with GPU support"
category: POST
tags: [JAX, T5x, jaxlib, jax, google, cudnn, nvcc, cuda, gpu]
---

Calculation of the derivatives plays a significant role in neural networks tuning. 
The computational effectivenes is very crusial considering a large models and ability to train them.
Besides the `CPU` and one of the most common option for calculations, recent advances finds a significant application of `GPU` and `TPUs`
mostly because of a potentially greater performance vs. the central processing unit.
Obvioulsy, such features would be available without a special software support, networks compilation towards the targeted device and platform.

<!--more-->

`JAX` is a library, proposed and maintained by Google which combines 
[autograd]()
and [XLA]() compiler in order to bring the most required operations related to neural networks onto the computational devices in a most efficient way.
However, speaking about `GPU` and more nn oriented devices as `TPU`, it is pretty important to perform a proper library setup in order to 
make them available for `JAX`.


Let's say we have installed:
```
jax==0.3.21
jaxlib==0.3.15
```

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
