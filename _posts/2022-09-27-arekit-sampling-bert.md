---
layout: post
title: "AREkit Tutorial: Sample Mass-Media Text Opinions for BERT"
description: "AREkit Tutorial: Sample Mass-Media Text Opinions for BERT"
category: POST
visible: 0
tags: [AREkit, Samples, Neural Networks, BERT]
---

<!--more-->

> **NOTE:** This post represents an updated version of the prior one
>["Process Mass-Media relations for Language Models with AREkit"](https://nicolay-r.github.io/blog/articles/2022-05/process-mass-media-relations-with-arekit)
; in the prior one we describe sampling process from scratch and under older API version *AREkit-0.22.0*.
>
## Sampler Initialization

First, it is necessary to declare labels expected to adopted in further samples preparation process.
In this post we focused on sentiment-related data sampling and therefore considering the following 
set of labels: `Positive`, `Negative` and additionally *neutral*, type of `NoLabel` which AREkit provides by default.
```python
class Positive(Label):
    pass

class Negative(Label):
    pass
```

Next step, we declare label scaler.
*Scaler* (`BaseLabelScaler` class) allows us to provide conversion from `Label` type to `int`/`uint` values and vice versa.
We declare Sentiment scaller as follows:
```python
class SentimentLabelScaler(BaseLabelScaler):
    def __init__(self):
        int_to_label = OrderedDict([(NoLabel(), 0), (Positive(), 1), (Negative(), -1)])
        uint_to_label = OrderedDict([(NoLabel(), 0), (Positive(), 1), (Negative(), 2)])
        super(SentimentLabelScaler, self).__init__(int_to_label, uint_to_label)
```

In terms of the input aspects of the **BERT** model, 
we deal with a sequence (optionally) separated by a `[SEP]` token onto couple parts, such as:
`TextA` and `TextB`.
For the classificational task, `TextB` might be treated as a prompt with the auxilary information 
which might be considered in a result class decission.
> For the sentiment analysis and relation extraction domain you may examine more approaches in 
[Awesome Sentiment Attitude Extraction Repository](https://github.com/nicolay-r/awesome-sentiment-attitude-extraction)

At present, `text_b` template is expected to contain a placeholders for `subject`, `object` and `context`,
where *context* corresponds to a text part between `subject` and `object`.
For texts in Russian, we assign the following **NLI-styled** (Natural Language Inference) prompt:
> NOTE: you may left `text_b_tempalete` as **None** once you don't want to consider a separated sequence.
```python
text_b_template = '{subject} к {object} в контексте : << {context} >>'
```

Next, we focused on text provider.
First, there is a need to setup terms mapper.
Terms mappers allows us to customize the way on how terms will be displayed in samples.
AREkit provides `BertDefaultStringTextTermsMapper`, in which you may among all 
of the different term types customize mentioned named entities. 

In terms of the latter we have a separated post 
[AREkit Tutorial: Entity Values Formatting Examples](https://nicolay-r.github.io/blog/articles/2022-09/arekit-entity-formatters-examples).
From that tutorial, here we adopt `CustomEntitiesFormatter` and assign `#S` and `#O` masks towards the
text opinion participants, i.e. subject and object respectively.

Depending on the `text_b_template` we may declare a single text provider (i.e. `TextA` only)
or pair-based one:
```python
terms_mapper = BertDefaultStringTextTermsMapper(
    entity_formatter=CustomEntitiesFormatter(
        subject_fmt="#S", object_fmt="#O"))

text_provider = BaseSingleTextProvider(terms_mapper) \
    if text_b_template is None else \
        PairTextProvider(text_b_template, terms_mapper)
```

Finally we may compose sample rows provider:
```python
sample_rows_provider = BaseSampleRowProvider(
    label_provider=MultipleLabelProvider(SentimentLabelScaler()),
    text_provider=text_provider)
```

Initialize information related to the samples format and output directory/path.
As for format, there is a need to declare a type inherited from the `BaseWriter`.
By default, AREkit provides `TsvWriter` -- is a `CSV`-style formatter.

> **Side note:** Tilte prefix `tsv` comes from the format proposed by google-BERT.
 
```python
writer = TsvWriter(write_header=True)
samples_io = SamplesIO("out/", writer, target_extension=".tsv.gz")
```

```python
pipeline_item = BertExperimentInputSerializerPipelineItem(
    sample_rows_provider=sample_rows_provider,
    samples_io=samples_io,
    save_labels_func=lambda data_type: True,
    balance_func=lambda data_type: data_type == DataType.Train)
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
The contents depend on Data Folding format.
For example, in case of the *fixed* folding onto `Train` and `Test` data types,
it is expected to see the following set of contents:
```
./out/
    sample_train.tsv.gz
    sample_test.tsv.gz
```
