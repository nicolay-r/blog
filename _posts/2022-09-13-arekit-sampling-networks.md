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

## Sampler Initialization

For such conventional neural networks as Convolutional NN's, Recurrent NN's,
it is expected that we manually provide input vectors (embeddings), 
including vocabulary of the supported words.
Therefore there is a need at first initialize the auxilary structures in order 
to then adopt them in sampler initialization.

Starting with an embedding preparation.
```python
stemmer = MystemWrapper()
embedding = load_embedding_news_mystem_skipgram_1000_20_2015(stemmer)
```

Label scaler
```python
class PositiveTo(Label):
    pass

class NegativeTo(Label):
    pass

class SentimentLabelScaler(BaseLabelScaler):
    def __init__(self):
        int_to_label = OrderedDict([
            (NoLabel(), 0), (PositiveTo(), 1), (NegativeTo(), -1)])
        uint_to_label = OrderedDict([
            (NoLabel(), 0), (PositiveTo(), 1), (NegativeTo(), 2)])
        super(SentimentLabelScaler, self).__init__(
            int_dict=int_to_label, uint_dict=uint_to_label)
```

Network serialization context -- is a structure of the data required during sampling.
This structure covers such further neural network features as:
* Part-of-Speech tagging
* Frames annotations 
* Frames connotations

In terms of frames connotations, we are kindly refer to our another post
[AREkit Tutorial: Frame Variants and Connotation Providers](https://nicolay-r.github.io/blog/articles/2022-09/arekit-frames)
in which we cover frame-based providers (Russian resource based only).
 
```python
ctx = CustomNetworkSerializationContext(
    labels_scaler=SentimentLabelScaler(),
    pos_tagger=POSMystemWrapper(mystem=stemmer.MystemInstance),
    frames_collection=frames_collection,
    frame_variant_collection=frame_variant_collection,
    frames_connotation_provider=RuSentiFramesConnotationProvider(frames_collection))
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

Entity formatter. 

> TODO. Organize a separated topic for so.

```python
class OpinionEntityType(Enum):
    Object = 1
    Subject = 2
    SynonymSubject = 3
    SynonymObject = 4
    Other = 5

class StringEntitiesFormatter(object):
    def to_string(self, original_value, entity_type):
        assert(isinstance(entity_type, OpinionEntityType))
        raise NotImplementedError()

class CustomEntitiesFormatter(StringEntitiesFormatter):

    def to_string(self, original_value, entity_type):
        if entity_type == OpinionEntityType.Other:
            return original_value
        elif entity_type == OpinionEntityType.Object or \
             entity_type == OpinionEntityType.SynonymObject:
            return "[object]"
        elif entity_type == OpinionEntityType.Subject or \
             entity_type == OpinionEntityType.SynonymSubject:
            return "[subject]"
        return None
```

And now, we are finally ready to compose our serializer. 
The latter represents a pipeline item, -- is an object which might be embedded into AREkit pipelines.

```python
pipeline_item = NetworksInputSerializerPipelineItem(
    vectorizers, samples_io, embedding_io,
    str_entity_fmt=entities_fmt,
    exp_ctx=ctx,
    balance_func=lambda data_type: data_type == DataType.Train,
    save_labels_func=lambda data_type: data_type != DataType.Test,
    save_embedding=True)
```

## Running Sampler

Please refer to the following posts in order to initialize your text opinion annotation pipeline (`annot_pipeline`)
and setup Data Folding (`data_folding`):
* [Craft your text-opinion annotation pipeline!](https://nicolay-r.github.io/blog/articles/2022-08/arekit-text-opinion-annotation-pipeline)
* [Data Folding Setup](https://nicolay-r.github.io/blog/articles/2022-09/arekit-sampling)

Finally, we can compose pipeline by wrapping a predefined `pipeline_item` and then run it!
This could be accomplished as follows:
```python
pipeline = BasePipeline([
    pipeline_item
])

pipeline.run(input_data=None,
             params_dict={
                 "data_folding": data_folding,
                 "data_type_pipelines": annot_pipeline 
             })
```

Finally our result is a content of the `out` directory.
The contents depends on Data Folding format.
For example, in case of the *fixed* folding onto `Train` and `Test` data types,
it is expected to see the following set of contents:
```
./out/
    sample_train.tsv.gz
    sample_test.tsv.gz
    embedding.npz
    vocab.txt
```
