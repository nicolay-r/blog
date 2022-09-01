---
layout: post
title: "AREkit Tutorial: Frame Variants and Connotation Providers"
description: "AREkit Tutorial: Frame Variants and Connotation Providers"
category: POST
visible: 0
tags: [AREkit, Frames, Connotations, RuSentiFrames]
---

![alt text](https://raw.githubusercontent.com/nicolay-r/blog/master/img/arekit-frames.png)

<!--more-->

Core component of the AREkit-0.22.1 framework is a `FrameVariant`.
The **Variant** suffix here denotes a one potential form of entry which stands behind the particular 
frame identifier.
This entry contains list of terms (`terms`) and its identifier (`frame_id`):

```python
class FrameVariant(object):
    def __init__(self, text, frame_id):
        assert(isinstance(text, str))
        assert(isinstance(frame_id, str))
        self.__terms = text.lower().split()
        self.__frame_id = frame_id
```

Class that provides implementation of gathering frame variants into collection 
 is called `FrameVariantsCollection`.
 
```python
class FrameVariantsCollection(object):
    def __init__(self):
        self.__variants = {}
        self.__frames_list = []

    def fill_from_iterable(self, variants_with_id, ...):
        # .. implementations out of this post.
        pass
```

## RuSentiFrames -- Frame Variants Provider

[RuSentiFrames](https://github.com/nicolay-r/RuSentiFrames) Represents a lexicon which describes sentiments and connotations conveyed with a predicate in a verbal or nominal form. 
Checkout the related [paper](https://aclanthology.org/R19-1118/) for greater details.
AREkit supports this frames collection out-of-the-box.

First, it is necessary to declare labels. 
We consider the following types:
`Positive`, `Negative`.

```python
class Positive(Label):
    pass

class Negative(Label):
    pass
```

Next step, we declare frames collection:

```python
frames_collection = RuSentiFramesCollection.read_collection(
    version=RuSentiFramesVersions.V20,
    labels_fmt=RuSentiFramesLabelsFormatter(
        pos_label_type=Positive, neg_label_type=Negative),
    effect_labels_fmt=RuSentiFramesEffectLabelsFormatter(
        pos_label_type=Positive, neg_label_type=Negative))
```

Then, variant collection might be initialized as follows:
```python
frame_variant_collection = FrameVariantsCollection()
frame_variant_collection.fill_from_iterable(
    variants_with_id=frames_collection.iter_frame_id_and_variants(),
    overwrite_existed_variant=True,
    raise_error_on_existed_variant=False)
```

Frame variant collection might be adopted in text parsing.
Checkout [AREkit Tutorial: Compose your text-processing pipeline!](https://nicolay-r.github.io/blog/articles/2022-08/arekit-text-parsing-pipeline) for a greater details.

## Connotation Provider

In such cases when the only information required from frames is their connections between 
roles (Agent, Theme), the related **connotation provider** finds its application.
Connotation provider could be presented in a form of the class or interface with 
a single method that provides such connotation-related information by a given `frame_id`.
Snippet below illustrates an AREkit-0.22.1 implementation of the base `FrameConnotationProvider`.

```python
class FrameConnotationProvider(object):
    def try_provide(self, frame_id):
        raise NotImplementedError()
```

Here is an example of the connotation provider based on the RuSentiFrames collection:
```python
class RuSentiFramesConnotationProvider(FrameConnotationProvider):
    def __init__(self, collection):
        self.__collection = collection
    def try_provide(self, frame_id):
        return self.__collection.try_get_frame_polarity(
            frame_id, role_src='a0', role_dest='a1')

frames_connotation_provider = RuSentiFramesConnotationProvider(frames_collection)
```

Connotation provider might be adopted in samples preparation for neural networks.
Checkout [AREkit Tutorial: Sample Mass-Media Text Opinions for Neural Network](https://nicolay-r.github.io/blog/articles/2022-09/arekit-sampling-networks)
for a greater details.
