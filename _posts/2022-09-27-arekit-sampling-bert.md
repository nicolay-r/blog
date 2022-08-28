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

> **TODO:** Sample Rows Provider. Provide class impelemntation with Pair-Based row provider.
 
```python
sample_rows_provider = ...
```

Initialize information related to the samples format and output directory/path.
As for format, there is a need to declare a type inherited from the `BaseWriter`.
By default, AREkit provides `TsvWriter` -- is a CSV-style formatter.

> **Side note:** Tilte prefix `tsv` comes from the format proposed by google-BERT.
 
```python
writer = TsvWriter(write_header=True)
samples_io = SamplesIO("out/", writer, target_extension=".tsv.gz")
```

```python
pipeline_item = BertExperimentInputSerializerPipelineItem(
    sample_rows_provider=None,
    samples_io=samples_io,
    save_labes_func=lambda data_type: True,
    balance_func=lambda data_type: data_type == DataType.Train
)
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
