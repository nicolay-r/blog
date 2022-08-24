---
layout: post
title: "AREkit Tutorial: Craft your text-opinion annotation pipeline!"
description: "AREkit Tutorial: Craft your text-opinion annotation pipeline!"
category: POST
tags: [Pipelines, Opinions, Attitudes, Relation Extraction, Relations, AREkit, RuSentRel, NEREL]
---

![alt text](https://raw.githubusercontent.com/nicolay-r/blog/master/img/arekit-text-opinion-annotation-pipeline.png)

In this post we are focusing on the relations extraction between a couple mentioned named entities in text.
Speaking precisely, we consider sentiment connections between mentioned named entities in text, i.e. `positive` or `negative` (and additionally `neutral`)
In order to automatically extract relations of such type, it is necessary to provide their markup in texts first, -- 
an algorithm, which allows us to extract text parts with potential connections between mentioned named entities.
The result markup could be then used for samples generation -- data, which is requred from Machine Learning model training, inferring, testing and so on.

<!--more-->

The snippet below illustrates the core function `text_opinion_extraction_pipeline`, pipeline, which allows us to 

```python
pipeline = text_opinion_extraction_pipeline(
    annotators=[
        # LIST OF OUR ANNOTATIONS
    ],
    text_opinion_filters=[
        # LIST OF OUR FILTERS
    ],
    get_doc_func=lambda doc_id: doc_ops.get_doc(doc_id),
    text_parser=text_parser)
```            

According to the snippet above, first of all we deal with **annotators** -- implmentations which provides iterators of the text opinions.
To accomplish this task, AREkit-0.22.1 provides the following annotators
* `PredefinedTextOpinionAnnotator` --
* `AlgorithmBasedOpinionAnnotator` -- 
* `AlgorithmBasedTextOpinionAnnotator` -- 

Another parameter in the snippet above is a set of text-opinion **filters**, which allows us to declare boolean rules in order reject some of text opoinions
depending on the certaion limitations, our needs, and so on.
AREkit-0.22.1 provides the following filters for text opinions by default:
* `EntityBasedTextOpinionFilter` --
* `DistanceLimitedTextOpinionFilter` --
* `FrameworkLimitationsTextOpinionFilter` -- 

> **NOTE** -- `FrameworkLimitationsTextOpinionFilter` is an internal limitations which is applied by default so there is no need to declare them 
manually, but it might be better to consider already known limitations for your personal needs.

In order to deal with data and prove the related docuemnts, it is necessary to declare `DocumentOperations` interface.
This interface has the following description and provides a single `get_doc` method, for a known document identifier:

```python
class DocumentOperations(object):
    def get_doc(self, doc_id):
        raise NotImplementedError()
```

Our blog already covers the question on 
[how the external collection might be binded in AREkit](https://nicolay-r.github.io/blog/articles/2022-08/arekit-collection-bind).
You may refer to that post for a greater details on how reader might be implemented for a `foo` example collection. 
The snippet below illustrates on how the related reader might be wrapped into DocumentOperations

```python
class DocumentOperations(object):
    def get_doc(self, doc_id):
        # Provide document reader.
        return FooDocReader.read_document(str(doc_id), doc_id=doc_id)
```

At last, as for the `text_parser` parameter, we also have a post which covers it in a greater details.

## Text Opinion Annotators

Let's take a closer look on how each of them might be, first **declared** and, second, 
**adopted** in the final in the common text opinion extraction pipeline. 
In this section we cover all the possible annontators that might be crafter out-of-the-box. 

Simple predefined annotator for the case, when we already have annotated text opinions 
(like BRAT-based collections, which provide so). 
For example, for texts in Russian, this might be a NEREL collection.

```python
predefined_annotator PredefinedTextOpinionAnnotator(
    doc_ops=doc_ops, 
    label_formatter=RuAttitudesLabelFormatter(label_scaler))
```

Sometimes, the annotation might be provided on document level, like in [RuSentRel].
```python
predefined_annotator = AlgorithmBasedTextOpinionAnnotator(
        annot_algo=PredefinedOpinionAnnotationAlgorithm(
            lambda doc_id: __get_document_opinions(doc_id, synonyms, labels_fmt)),
        create_empty_collection_func=lambda: OpinionCollection(
            opinions=[], synonyms=synonyms, 
            error_on_duplicates=True, 
            error_on_synonym_end_missed=False),
        get_doc_existed_opinions_func=lambda _: None,
        value_to_group_id_func=lambda value:
            SynonymsCollectionValuesGroupingProviders.provide_existed_value(synonyms, value))

def __get_document_opinions(doc_id, synonyms, labels_fmt):
    return OpinionCollection(
        opinions=RuSentRelOpinionCollection.iter_opinions_from_doc(doc_id, labels_fmt),
        synonyms=synonyms,
        error_on_synonym_end_missed=True,
        error_on_duplicates=True)
```

No label algorithm-based opinion annotator. By default, AREkit-0.22.1 provides a `NoLabel` instance.

```python
nolabel_annotator = AlgorithmBasedTextOpinionAnnotator(
    annot_algo=PairBasedOpinionAnnotationAlgorithm(
        dist_in_sents=dist_in_sentences,
        dist_in_terms_bound=terms_per_context,
        label_provider=ConstantLabelProvider(NoLabel())),
    create_empty_collection_func=lambda: OpinionCollection(
        opinions=[], synonyms=synonyms, error_on_duplicates=True, error_on_synonym_end_missed=False),
    get_doc_existed_opinions_func=lambda _: None,
        value_to_group_id_func=lambda value:
            SynonymsCollectionValuesGroupingProviders.provide_existed_value(
                synonyms=synonyms, value=value))
```

# Text Opinions Filters

In this section we provide the details on which filters might be declared.

The base class for the opinion filter declaration has the following API:
> **NOTE**: We consier entity service by default in order to accelerate filtering process since
this service is the most required in filtering across the predefined filters.
```python
class TextOpinionFilter(object):
    def filter(self, text_opinion, parsed_news, entity_service_provider):
        raise NotImplementedError()
```

```python
EntityBasedTextOpinionFilter(entity_filter)
```

In the snippet below, we consider only those text opinions, in which distance in terms between participats is not exceeds amount of `50`:
```python
DistanceLimitedTextOpinionFilter(terms_per_context=50)
```

# Result Pipeline
