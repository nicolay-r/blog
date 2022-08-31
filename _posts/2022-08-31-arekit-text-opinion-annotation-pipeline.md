---
layout: post
title: "AREkit Tutorial: Craft your text-opinion annotation pipeline!"
description: "AREkit Tutorial: Craft your text-opinion annotation pipeline!"
category: POST
visible: 0
tags: [Pipelines, Opinions, Attitudes, Relation Extraction, Relations, AREkit, RuSentRel, NEREL]
---

![alt text](https://raw.githubusercontent.com/nicolay-r/blog/master/img/arekit-text-opinion-annotation-pipeline.png)

In this post we are focusing on the relations extraction between a couple mentioned named entities in text.
Speaking precisely, we consider sentiment connections between mentioned named entities in text, i.e. `positive` or `negative` (and additionally `neutral`)
In order to automatically extract relations of such type, it is necessary to provide their markup in texts first, -- 
an algorithm, which allows us to extract text parts with potential connections between mentioned named entities.
The result markup could be then used for samples generation -- data, which is requred from Machine Learning model training, inferring, testing and so on.

<!--more-->

> [Example code for the tutorial](https://github.com/nicolay-r/AREkit/blob/af7c951d871a693a25a0a7c72e8a0ff5ba559c4a/tests/tutorials/test_tutorial_pipeline_text_opinion_annotation.py#L43)

The snippet below illustrates the core function `text_opinion_extraction_pipeline`, pipeline, which allows us to 

```python
pipeline = text_opinion_extraction_pipeline(
    annotators=[
        # LIST OF YOUR ANNOTATIONS
    ],
    text_opinion_filters=[
        # LIST OF YOUR FILTERS
    ],
    get_doc_func=lambda doc_id: doc_ops.get_doc(doc_id),
    text_parser=text_parser)
```            

According to the snippet above, first of all we deal with **annotators** -- implmentations which provides iterators of the text opinions.
To accomplish this task, AREkit-0.22.1 provides the following annotators
1. `PredefinedTextOpinionAnnotator` -- consider to convert BRAT-based relations into text opinions
2. `AlgorithmBasedOpinionAnnotator` -- consider to adopt algoritms for opinion annotation onto the document level.
3. `AlgorithmBasedTextOpinionAnnotator` -- the same as (2), but with a conversion onto text-level; this is inherited from the (2) in AREkit-0.22.1;

Another parameter in the snippet above is a set of text-opinion **filters**, which allows us to declare 
boolean rules in order reject some of text opoinions depending on the certaion limitations, our needs, and so on.
By default, AREkit-0.22.1 provides the following filters for text opinions:
* `EntityBasedTextOpinionFilter` -- filter based on ends of the text opinion participants (subject and object named entities)
* `DistanceLimitedTextOpinionFilter` -- filter based on distance between mentioned named entities (in terms)
* `FrameworkLimitationsTextOpinionFilter` -- limitations that should be considered due to specifics of the internal fucntionality implementation.

> **NOTE**: `FrameworkLimitationsTextOpinionFilter` is an internal limitations which is applied by default so there is no need to declare them 
manually, but it might be better to consider already known limitations for your personal needs.

In order to deal with data and prove the related docuemnts, it is necessary to declare `DocumentOperations` interface.
This interface has the following description and provides a single `get_doc` method, for a known document identifier:

```python
class DocumentOperations(object):
    def get_doc(self, doc_id):
        raise NotImplementedError()
```

Our blog already covers the question on 
[AREkit Tutorial: Binding a custom annotated collection for Relation Extraction](https://nicolay-r.github.io/blog/articles/2022-08/arekit-collection-bind).
You may refer to that post for a greater details on how reader might be implemented for a `foo` example collection. 
The snippet below illustrates on how the related reader might be wrapped into DocumentOperations

```python
class FooDocumentOperations(DocumentOperations):
    def get_doc(self, doc_id):
        return FooDocReader.read_document(str(doc_id), doc_id=doc_id)
```

At last, as for the `text_parser` parameter, we also have a post which covers it in a greater details.
Please refer to the [AREkit Tutorial: Compose your text-processing pipeline!](https://nicolay-r.github.io/blog/articles/2022-08/arekit-text-parsing-pipeline) for the related details.

## Text Opinion Annotators

Let's take a closer look on how each of them might be, first **declared** and, second, 
**adopted** in the final in the common text opinion extraction pipeline. 
In this section we cover all the possible annontators that might be crafter out-of-the-box.

Before we start with annotators, the common formatter we also required is a **labels formatter**.
To implement your own formatter, it is important to inherit `StringLabelsFormatter` base AREkit-0.22.1 class.
Labels formatter allows us to perform transformation from `str` to `Label` type and vice versa. 
Type `Label` is a core type for every label utilzed in a project.
The snippet below illustrates an example of the custom label formatter for a couple labels, such as `positive` and `negative`:

```python
class CustomLabelsFormatter(StringLabelsFormatter):
    def __init__(self, pos_label_type, neg_label_type):
        stol = {"neg": neg_label_type, "pos": pos_label_type}
        super(CustomLabelsFormatter, self).__init__(stol=stol)
```

Simple predefined annotator for the case, when we already have annotated text opinions (like BRAT-based collections). 
For texts in Russian, this might be a [NEREL collection](https://github.com/nerel-ds/NEREL).
The snipped below illstrates on how the annotator of the predefined relations might be implemented:

```python
class PositiveLabel(Label):
    pass

class NegativeLabel(Label):
    pass

predefined_annotator = PredefinedTextOpinionAnnotator(
    doc_ops=doc_ops,
    label_formatter=CustomLabelsFormatter(pos_label_type=PositiveLabel,
                                          neg_label_type=NegativeLabel))
```

Sometimes, the annotation might be provided on document level, like in [RuSentRel](https://github.com/nicolay-r/RuSentRel).
The snippet below illustrates the way on how document level opinions, provided separately for every document of RuSentRel collection,
might be adopted and then converted to text opinions. 
This conversion is performed via `RuSentRelOpinionCollection.iter_opinions_from_doc` 
which leave outside of this post.

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

No label algorithm-based opinion annotator. 
As for the snippet above, it will be also required a synonyms collection,
which is a `StemmerBasedSynonymCollection` in our case by default, based on the Yandex Mystem stemmer.
By default, AREkit-0.22.1 provides a `NoLabel` instance.

```python
synonyms = StemmerBasedSynonymCollection(
    iter_group_values_lists=[], stemmer=MystemWrapper(), is_read_only=False, debug=False)

nolabel_annotator = AlgorithmBasedTextOpinionAnnotator(
    annot_algo=PairBasedOpinionAnnotationAlgorithm(
        dist_in_sents=0,
        dist_in_terms_bound=50,
        label_provider=ConstantLabelProvider(NoLabel())),
    create_empty_collection_func=lambda: OpinionCollection(
        opinions=[], synonyms=synonyms, error_on_duplicates=True, error_on_synonym_end_missed=False),
    get_doc_existed_opinions_func=lambda _: None,
        value_to_group_id_func=lambda value:
            SynonymsCollectionValuesGroupingProviders.provide_existed_value(synonyms, value))
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

Where the its custom implementation might be as follows:

```python
class CustomEntityFilter(EntityFilter):
    supported = ["GPE", "PERSON", "LOCAL", "GEO", "ORG"]
    def is_ignored(self, entity, e_type):
        if e_type == OpinionEntityType.Subject or e_type == OpinionEntityType.Object:
            return entity.Type not in CustomEntityFilter.supported
        return True
```

Here is how the filter, based on the entity filtering details, might be gathered:

```python
custom_filter = EntityBasedTextOpinionFilter(entity_filter=CustomEntityFilter())
```

In the snippet below, we consider only those text opinions, 
in which distance in terms between participats is not exceeds amount of `50`:

```python
distance_filter = DistanceLimitedTextOpinionFilter(terms_per_context=50)
```

# Conclusion

> [Example code for the tutorial](https://github.com/nicolay-r/AREkit/blob/af7c951d871a693a25a0a7c72e8a0ff5ba559c4a/tests/tutorials/test_tutorial_pipeline_text_opinion_annotation.py#L43)

Thank you for reading this post! 
Now you're in details on how text opinion annotators and filters might be implemented in order to craft your own
text opinion annotator. 
Annotator then is requred for data sampling, which will be covered in next posts in greater details. 
