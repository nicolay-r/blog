---
layout: post
title: "AREkit Tutorial: Compose your text-processing pipeline!"
description: "AREkit Tutorial: Compose your text-processing pipeline!"
category: POST
tags: [Pipelines, Frames, Tokenization, Named Entity Recognition, NER, AREkit]
---

![alt text](https://raw.githubusercontent.com/nicolay-r/blog/master/img/arekit-text-parsing.png)


In [AREkit-0.22.1](https://github.com/nicolay-r/AREkit), 
we provide `BaseTextParser` which assumes to apply a series of text processing items, organized in a form of the `PipelineItem`'s,
in order to modify the text contents with the annotated objects in it. 
The common class which is related to performing transformation from the original `News` towards the processed one is a `BaseTextParser`, which receives
a pipeline of the **annotations** expected to be applied towards a news text:

<!--more-->

```python
text_parser = BaseTextParser(pipeline=[ 
    # ... list of the pipeline items that is related to the text processsing 
    # (exploring them in details within the next sections of this post)
])
parsed_news = NewsParser.parse(news, text_parser)
```
Where type `ParsedNews` represents a processed version of the text, where every news sentence might inlude annotated objects, such as: 
* `Entity` -- mentioned named entity in text.
* `Token` -- punctuation sign, URL-links, number, etc.
* `FrameVarian` -- word that is presented in the external Frame-based lexicons.
* Ordinary `str` entries
* Any other manually declared types.

All these types are required for **Services** that might be built on top of the parsed news. We decided to leave the details about them outside of this post.
Within next sections we focusing on the Pipeline items that could be applicable for text processing.

## Entities Annotation

Among all of the potential annotations that could be applied towards text, 
the most important one is related to the **annotation of the mentioned named entities** (NE) in text.
This task might be performed manually and provided as part of the document annotation itself, or 
the latter might be accomplished automatically. 
We may encounter with the predefined annotation once we treating the BRAT-based collections, 
or other collections that provides such annotation by default and performed by the 
[embedded](https://github.com/nicolay-r/AREkit/blob/629ee6d2705980b4a7ad792faa3f7baae5b57973/arekit/contrib/source/brat/entities/parser.py#L8) 
`BratTextEntitiesParser`.

> **NOTE:** We may consider a different partitioning formats of the original text of `News` instance:
`string` for the case when every sentence of the news represented in a form of the strings, and
`terms` when we deal with list of tokens as a contents of every sentence.

```python
text_parser = BaseTextParser([
    BratTextEntitiesParser(partitioning="string")
])
```
We may also deal with texts, in which entities are annotated by keeping so in a square brackets.
For such cases you may adopt the [following](https://github.com/nicolay-r/ARElight/blob/74d424b38589fe5038518229a17ca32f2dd97867/arelight/text/pipeline_entities_default.py#L5) parser:
> NOTE: This parser is a part of the side project [ARElight](https://github.com/nicolay-r/ARElight)

```python
text_parser = BaseTextParser([
    # ... 
    TextEntitiesParser(),
])
```

In case of a raw texts we deal with the NER task, dubbed as *Named Entity Recognition* problem.
The snippet below illustrates on how the `BERT_ontonotes` model could be adopted for an automatic 
named entities annotation ([see the details](https://github.com/nicolay-r/ARElight/blob/74d424b38589fe5038518229a17ca32f2dd97867/arelight/text/pipeline_entities_bert_ontonotes.py#L9) 
of the `BertOntonotesNERPipelineItem` implementation):
> **NOTE:** We cover this scenario in a greater deails as a part of the 
[ARElight](https://github.com/nicolay-r/ARElight/blob/v0.22.1/arelight/text/ner_ontonotes.py) project
by adopting BERT models for NER using [DeepPavlov](https://deeppavlov.ai/) framework (`BertOntonotesNER` class); 
we left so outside of the following post and kindly refer you to the related project.

```python
text_parser = BaseTextParser([
    # considering to apply BERT-ontonotes model and pick only specific object types.
    BertOntonotesNERPipelineItem(lambda s_obj: s_obj.ObjectType in ["ORG", "PERSON", "LOC", "GPE"])
])
```

## Tokens and Terms Annotation

Besides the mentioned named entities itself, mostly there is a need to separate words from each other.
In siple case `TesrmsSplitterParser` allows to perform separation by a known separator (whitespaces by default),
and declare so as follows (see class [implementation](https://github.com/nicolay-r/AREkit/blob/629ee6d2705980b4a7ad792faa3f7baae5b57973/arekit/contrib/utils/pipelines/items/text/terms_splitter.py#L6) 
for a greater details)

```python
text_parser = BaseTextParser([
    # ... 
    TermsSplitterParser(keep_tokens=True),
])
```

For a detailed analysis, we treat this stage as a *tokenization* process. 
[AREkit-0.22.1](https://github.com/nicolay-r/AREkit) provides a `DefaultTextTokenizer` for so. 
This tokenizer allows us to demarcate words from such text constructions as: 
* Punctuation signs 
* URL-links
* Numbers

```python
text_parser = BaseTextParser([
    # ... 
    DefaultTextTokenizer(keep_tokens=True),
])
```

## Frames Annotation

*Frames*, -- is a certain text words or prases that may emphasize the presence of the relation
Frames are useful in certain Machine Learning models, designed to solve Relation Extraction problems.
In terms of the such task as *Sentiment Analysis*, frames might be entries that convey the presence of the sentiment attutdies from one object towards the other.

In AREkit-0.22.1 we provide [declaration](https://github.com/nicolay-r/AREkit/blob/629ee6d2705980b4a7ad792faa3f7baae5b57973/arekit/common/frames/variants/collection.py#L5) of the `FramesVariantsCollection`'s. 
This collection allows keeping frame variants (`FrameVariant`) for a given frame ID.
For studies in Russian we provide `RuSentiFramesCollection` which provides connotation frames that conveys the presence of sentiment relations from Agent (`A0`) towards Theme (`A1`) with such sentimnets as: *positive* (`PositiveTo` in the following example) and *negative* (`NegativeTo`). 
Frames collection initialization is not a part of the following post so we kindly refer
you to another 
[Frame Variants and Connotation Providers](https://nicolay-r.github.io/blog/articles/2022-09/arekit-frames)
tutorial for a greater details onto frames concepts and providers initialization.

Then, application of the frame variants annotation could be adopted as follows:
> **NOTE**: for `frame_variant_collection` initialization please follow the details of the
[AREkit Tutorial: Frame Variants and Connotation Providers](https://nicolay-r.github.io/blog/articles/2022-09/arekit-frames)
post.

```python
text_parser = BaseTextParser(pipeline=[
    # ...
    FrameVariantsParser(frame_variant_collection)
])
```

Due to the Russian texts, **lemmatization** might be required in analysis.
In terms of the Russian Language, AREkit provides the wrapper over [Yandex Mystem](https://yandex.ru/dev/mystem/) 
library, which could be adopted for terms lemmatization.
The snippet above could be modified with
`LemmaBasedFrameVariantsParser` as follows:
```python
text_parser = BaseTextParser(pipeline=[
    # ... lemmatized version
    LemmasBasedFrameVariantsParser(frame_variants=frame_variant_collection, 
                                   stemmer=MystemWrapper())
])
```

We may also adopt sentiment *negations* for frame variants, which allows us to invert sentiment score due to the particular prepositions.
> **NOTE:** This feature is a language specific, and in terms of the AREkit, for now available for Russian texts only.

```python
text_parser = BaseTextParser(pipeline=[
    # ... 
    FrameVariantsSentimentNegation()
])
```

## Gathering Your Own Text Processing Pipeline

Below is an example on how the items described above might be adopted for text processing:
```python
text_parser = BaseTextParser(pipeline=[
    TextEntitiesParser(),
    DefaultTextTokenizer(keep_tokens=True),
    LemmasBasedFrameVariantsParser(frame_variants=frame_variant_collection,
                                   stemmer=MystemWrapper()),
    FrameVariantsSentimentNegation()
])
```

## Conclusion

Congratulations! Now you're into details on how your custom text pipeline could be gathered!

Once we composed our text processing pipeline, we may then declare our *data-formatting pipeline* and describe on how we would like to annotated connection between text objects, applying filtering rules and declaring other restrictions on so. 
The latter is a part of our next posts so stay connected for a greater details.
