---
layout: post
title: "AREkit Tutorial: Sample Mass-Media Text Opinions for Your Machine Learning Model"
description: "AREkit Tutorial: Sample Mass-Media Text Opinions for Your Machine Learning Model"
category: POST
visible: 0
tags: [AREkit, CV, Split, Samples]
---

<!--more-->

> **NOTE:** This post represents an updated version of the prior one 
>["Process Mass-Media relations for Language Models with AREkit"](https://nicolay-r.github.io/blog/articles/2022-05/process-mass-media-relations-with-arekit)
; in the prior one we describe sampling process from scratch and under older API version *AREkit-0.22.0*.

In this post we consider that the prior AREkit tutorials tutorials were passed:
1. [Binding a custom annotated collection for Relation Extraction](https://nicolay-r.github.io/blog/articles/2022-08/arekit-collection-bind)
2. [Compose your text-processing pipeline!
](https://nicolay-r.github.io/blog/articles/2022-08/arekit-text-parsing-pipeline)
3. [Craft your text-opinion annotation pipeline!](https://nicolay-r.github.io/blog/articles/2022-08/arekit-text-opinion-annotation-pipeline)
4. [Data Folding Setup](https://nicolay-r.github.io/blog/articles/2022-09/arekit-sampling)

## Text Opinion Samplers

1. For conventional neural networks, i.e. **Convolutional NN's**, **Recurrent NN's**.


The result serializer represents a pipeline item, which could be gathered as follows:

Embedding preparation.
```python
stemmer = MystemWrapper()
embedding = load_embedding_news_mystem_skipgram_1000_20_2015(stemmer)
```

Network serialization context -- is a structure of the data required during sampling.
```python
ctx = CustomNetworkSerializationContext(
    labels_scaler=labels_scaler,
    pos_tagger=POSMystemWrapper(mystem=stemmer.MystemInstance),
    frames_collection=frames_collection,
    frame_variant_collection=frame_variant_collection)
```

**Vectorizers** -- algorithms of the vectors generation for text terms of any type.
AREkit provides the set of the following text **tokens**: word, entity, frame, token (punctuation sign, numbers, URL-links).
For each token type, it is possible to provide a custom vectorizer.
```python
bpe_vectorizer = BPEVectorizer(embedding=embedding, max_part_size=3)
norm_vectorizer = RandomNormalVectorizer(vector_size=embedding.VectorSize,
                                         token_offset=12345)
vectorizers = {
    TermTypes.WORD: bpe_vectorizer,
    TermTypes.ENTITY: bpe_vectorizer,
    TermTypes.FRAME: bpe_vectorizer,
    TermTypes.TOKEN: norm_vectorizer
}
```

Initialize information related to the samples format and output directory/path.
As for format, there is a need to declare a type inherited from the `BaseWriter`.
By default, AREkit provides `TsvWriter` -- is a CSV-style formatter.
> **Side note:** Tilte prefix `tsv` comes from the format proposed by google-BERT.
```python
writer = TsvWriter(write_header=True)
samples_io = SamplesIO("out/", writer, target_extension=".tsv.gz")
```

Besides the samples itself, it is necessary to provide the **formatting for embedding and vocabulary** related data.
By defualt, AREkit provides numpy-based and text encoders for storing embedding and vocabulary respectively.
`NpEmbeddingIO` provides such functionality, and therefore formatting based on the capabilities of the latter 
might be initialized as follows:
```python
embedding_io = NpEmbeddingIO(target_dir="out/")
```

And now, we are finally ready to compose our serializer. 
The latter represents a pipeline item, -- is an object which might be embedded into AREkit pipelines.

```python
pipeline_item = NetworksInputSerializerPipelineItem(
    vectorizers, samples_io, embedding_io,
    str_entity_fmt=entities_fmt,
    balance_func=lambda data_type: data_type == DataType.Train,
    save_labels_func=lambda data_type: data_type != DataType.Test,
    exp_ctx=ctx,
    save_embedding=True)
```

There is another post covers the sampling for BERT models.
