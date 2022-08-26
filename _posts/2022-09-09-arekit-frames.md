---
layout: post
title: "AREkit Tutorial: Frame Variants and Connotation Providers"
description: "AREkit Tutorial: Frame Variants and Connotation Providers"
category: POST
visible: 0
tags: [AREkit, Frames, Connotations, RuSentiFrames]
---

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

```python
class PositiveTo(Label):
    pass

class NegativeTo(Label):
    pass

frames_collection = RuSentiFramesCollection.read_collection(
    version=RuSentiFramesVersions.V20,
    labels_fmt=RuSentiFramesLabelsFormatter(
        pos_label_type=PositiveTo, neg_label_type=NegativeTo),
    effect_labels_fmt=RuSentiFramesEffectLabelsFormatter(
        pos_label_type=PositiveTo, neg_label_type=NegativeTo))
```

Then, variant collection might be initialized as follows:
```python
# Frame variant collection.
frame_variant_collection = FrameVariantsCollection()
frame_variant_collection.fill_from_iterable(
    variants_with_id=frames_collection.iter_frame_id_and_variants(),
    overwrite_existed_variant=True,
    raise_error_on_existed_variant=False)
```

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
