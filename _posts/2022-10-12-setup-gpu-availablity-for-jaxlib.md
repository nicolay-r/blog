---
layout: post
title: "Setup JAX framework with GPU support"
description: "Setup JAX framework with GPU support"
category: POST
tags: [JAX, T5x, jaxlib, jax, cudnn, nvcc, cuda, gpu]
---


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
