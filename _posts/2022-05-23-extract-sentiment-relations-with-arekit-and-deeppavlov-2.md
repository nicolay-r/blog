---
layout: post
title: "\"X raise sanctions against Y\" - Infer Sentiment Relations with AREkit and DeepPavlov [part 2/2]"
description: "\"X raise sanctions against Y\" - Infer Sentiment Relations with AREkit and DeepPavlov [part 1/2]"
category: POST
tags: [Sentiment Analysis, Relation Extraction, DeepPavlov, Finetunning, Language Models, BERT]
---

![alt text]({{site.url}}/img/arekit_deepPavlov-infer.png)

In this post we apply fine-tuned model for inferring sentiment relations from Mass-Media texts.

<!--more-->

> [COMPLETE EXAMPLE](https://github.com/nicolay-r/ARElight/blob/main/examples/infer_texts_bert.py)

The code snippets provided below are related to the:

* Serialization stage, organized by AREkit.
```python
BertTextsSerializationPipelineItem(
    synonyms=read_synonyms_collection(synonyms_filepath=synonyms_filepath, 
                                      stemmer=stemmer),
    terms_per_context=terms_per_context,
    entities_parser=BertOntonotesNERPipelineItem(
        lambda s_obj: s_obj.ObjectType in ["ORG", "PERSON", "LOC", "GPE"]),
    entity_fmt=create_entity_formatter(EntityFormatterTypes.HiddenBertStyled),
    name_provider=ExperimentNameProvider(name="example-bert", suffix="infer"),
    text_b_type=text_b_type,
    output_dir=output_dir,
    opin_annot=DefaultAnnotator(
        PairBasedAnnotationAlgorithm(
            dist_in_terms_bound=None,
            label_provider=ConstantLabelProvider(label_instance=NoLabel()))),
    data_folding=NoFolding(doc_ids_to_fold=list(range(texts_count)),
                           supported_data_types=[DataType.Test])),
```

* Sentiment Relation Extraction, provided by DeepPavlov library.
```python
BertInferencePipelineItem(
    data_type=DataType.Test,
    predict_writer=TsvPredictWriter(),
    bert_config_file=bert_config_path,
    model_checkpoint_path=bert_finetuned_ckpt_path,
    vocab_filepath=bert_vocab_path,
    max_seq_length=max_seq_length,
    do_lowercase=do_lowercase,
    labels_scaler=labels_scaler),
```

* Backend. Visualization, provided by Brat library.
```python
BratBackendContentsPipelineItem(label_to_rel={
    str(labels_scaler.label_to_uint(ExperimentPositiveLabel())): "POS",
    str(labels_scaler.label_to_uint(ExperimentNegativeLabel())): "NEG"
},
    obj_color_types={"ORG": '#7fa2ff', "GPE": "#7fa200", 
                     "PERSON": "#7f00ff", "Frame": "#00a2ff"},
    rel_color_types={"POS": "GREEN", "NEG": "RED"},
)
```



