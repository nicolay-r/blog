---
layout: post
title: "AREkit Tutorial: Sample Mass-Media Text Opinions for Neural Network"
description: "AREkit Tutorial: Sample Mass-Media Text Opinions for Neural Network"
category: POST
visible: 0
tags: [AREkit, Samples, Neural Networks, CNN, RNN]
---

![alt text](https://raw.githubusercontent.com/nicolay-r/blog/master/img/arekit-collection-sampling-nn.png)

In this tutorial we provide a list of steps required to prepare samples with text opinions for 
neural networks like convolutional or recurrent one.

<!--more-->

> [Code implementation](https://github.com/nicolay-r/AREkit/blob/6b0427201e41729f342046ec2261a0a2c6eda139/tests/tutorials/test_tutorial_pipeline_sampling_network.py#L52)

> **NOTE:** This post represents an updated version of the prior one 
>["Process Mass-Media relations for Language Models with AREkit"](https://nicolay-r.github.io/blog/articles/2022-05/process-mass-media-relations-with-arekit)
; in the prior one we describe sampling process from scratch and under older API version *AREkit-0.22.0*.

## Sampler Initialization

First, it is necessary to declare labels expected to adopted in further samples preparation process. 
In this post we focused on sentiment-related data sampling and therefore considering the following set of labels: 
`Positive`, `Negative` and additionally *neutral*, type of `NoLabel` which AREkit provides by default.

```python
class Positive(Label):
    pass

class Negative(Label):
    pass
```

Next step, we declare label scaler. 
Scaler (`BaseLabelScaler` class) allows us to provide conversion from `Label` type to `int`/`uint` values and vice versa. 
We declare Sentiment scaller as follows:
> **NOTE**: adopt `CustomSentimentLabelScaler` which is nested from the `SentimentLabelScaler` since the latter provides 
labels inversion `invert_label` required for frames (role label scaler); as an another solution is to provide a diffrent 
label scalers, however within this tutorial it was decide to adopt a single label scaler.

```python
class CustomSentimentLabelScaler(SentimentLabelScaler):
    def __init__(self):
        int_to_label = OrderedDict([(NoLabel(), 0), (Positive(), 1), (Negative(), -1)])
        uint_to_label = OrderedDict([(NoLabel(), 0), (Positive(), 1), (Negative(), 2)])
        super(SentimentLabelScaler, self).__init__(int_dict=int_to_label, uint_dict=uint_to_label)

    def invert_label(self, label):
        int_label = self.label_to_int(label)
        return self.int_to_label(-int_label)
```

For such conventional neural networks as Convolutional NN's, Recurrent NN's,
it is expected that we manually provide input vectors (embeddings), 
including vocabulary of the supported words.
Therefore there is a need at first initialize the auxilary structures in order 
to then adopt them in sampler initialization.

```python
stemmer = MystemWrapper()
embedding = load_embedding_news_mystem_skipgram_1000_20_2015(stemmer)
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
ctx = NetworkSerializationContext(
    labels_scaler=CustomSentimentLabelScaler(),
    pos_tagger=POSMystemWrapper(mystem=stemmer.MystemInstance),
    frame_roles_label_scaler=CustomSentimentLabelScaler(),
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

And now, we are finally ready to compose our serializer!
The latter represents a pipeline item, -- is an object which might be embedded into AREkit pipelines.

> **NOTE:** In a snippet below there is a need to declare formatter for **mentioned named entities**
(`str_entities_fmt` parameter).
In our blog we have a separated topic and post
[AREkit Tutorial: Entity Values Formatting Examples](https://nicolay-r.github.io/blog/articles/2022-09/arekit-sampling-networks);
in this example we adopt `SimpleUppercasedEntityFormatter` by default.

```python
pipeline_item = NetworksInputSerializerPipelineItem(
    vectorizers, samples_io, embedding_io,
    str_entity_fmt=SimpleUppercasedEntityFormatter(),
    exp_ctx=ctx,
    balance_func=lambda data_type: data_type == DataType.Train,
    save_labels_func=lambda data_type: data_type != DataType.Test,
    save_embedding=True)
```

## Running Sampler

> [Code implementation](https://github.com/nicolay-r/AREkit/blob/6b0427201e41729f342046ec2261a0a2c6eda139/tests/tutorials/test_tutorial_pipeline_sampling_network.py#L52)

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
The contents depend on Data Folding format.
For example, in case of the *fixed* folding onto `Train` and `Test` data types,
it is expected to see the following set of contents:

```
./out/
    sample_train.tsv.gz
    sample_test.tsv.gz
    embedding.npz
    vocab.txt
```
