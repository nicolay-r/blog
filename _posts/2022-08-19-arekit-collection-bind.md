---
layout: post
title: "AREkit Tutorial: Binding a custom annotated collection for Relation Extraction"
description: "AREkit Tutorial: Binding a custom annotated collection for Relation Extraction"
category: POST
tags: [Relation Extraction, BRAT, AREkit]
---

Source for annotation usually represent a raw text or provided with the bunch of annotations. The one of the most convinient way for creating and collaborative annotation ediding in Relation Extraction is a BRAT toolset. Besides the nice rendering and clear visualization of the all relations in text, it provides a web-based editor and ability to export the annotated data. However, the exported data is not prepared for most ML-based relation extraction models since it provides all the possible annotations for a single document. In order to simplify and structurize the contents onto text parts with the particular and fixed amount of annotations in it, in this post we propose the AREkit toolset and cover the API which provides an opportunity to bind your custom collection, based on BRAT annotation.

<!--more-->

> NOTE: We may adopt a raw texts, and the latter required a side application of NER. We remain this behind this post. However, for a greater details on this point you may proceed with the following project.

Lets get started ... Considering that we have a collection with the following structure of the `collection`, 
presented in a form of the archive:

```
collection.zip/
    1.txt
    1.ann
    ...
    xxx.txt
    xxx.ann
```

In this tutorial we rely on [AREkit-0.22.1](https://github.com/nicolay-r/AREkit).
Most of the API relies on the collection version.
Therefore it is required to provide the details onto versions that your collection support.
In this post we consider that our collection represents a `V1` version.
```python
class CollectionVersions(Enum):
    V1 = "V1"
```

Next, there is a need to establish connection between files and its contents.
Source-related contribution part provides API for interaction with contents in `zip` archives (class `ZipArchiveUtils`). 
To adopt the latter we provide an inherited class by declaring the following information: 
* archive filepath
* news and annotation filenames
```python
class CollectionIOUtils(ZipArchiveUtils):

    archive_path = "../data/sentiment_dataset.zip"

    @staticmethod
    def get_archive_filepath(version):
        return CollectionIOUtils.archive_path

    @staticmethod
    def get_annotation_innerpath(filename):
        return "{}.ann".format(filename)

    @staticmethod
    def get_news_innerpath(filename):
        return "{}.txt".format(filename)

    @staticmethod
    def __iter_filenames_from_dataset():
        for filename in CollectionIOUtils.iter_filenames_from_zip(CollectionVersions.V1):
            yield basename(filename)

    @staticmethod
    def iter_collection_filenames():
        filenames_it = CollectionIOUtils.__iter_filenames_from_dataset()
        for doc_id, filename in enumerate(filenames_it):
            yield doc_id, filename
```

Then since we deal with objects, mentioned in text, based on annotation files 
(`*.ann` extensions by default), 
there is a need to declare an `EntityCollection`.
This collection allows to access to entities by its values.
Some values might be sinonymous, i.e. different but related to a single entity.
Our collection does not provide information on how synonyms might be grouped and therefore, in the snippet
below we declare an empty and exapandable collection.
> **NOTE:** We adopt stemmer-based collection which supports grouping by lemmatized version of the given word; 
as for lemmatized AREkit provides a wrapper over Mystem (for Russian words)

```python
class CollectionEntityCollection(EntityCollection):

    def __init__(self, contents, value_to_group_id_func):
        super(CollectionEntityCollection, self).__init__(contents["entities"], value_to_group_id_func)
        self._sort_entities(key=lambda entity: entity.IndexBegin)

    @classmethod
    def read_collection(cls, filename, version):
        synonyms = StemmerBasedSynonymCollection(
            iter_group_values_lists=[], stemmer=MystemWrapper(), is_read_only=False, debug=False)
        return CollectionIOUtils.read_from_zip(
            inner_path=CollectionIOUtils.get_annotation_innerpath(filename),
            process_func=lambda input_file: cls(
                contents=BratAnnotationParser.parse_annotations(input_file),
                value_to_group_id_func=lambda value:
                    SynonymsCollectionValuesGroupingProviders.provide_existed_or_register_missed_value(
                        synonyms, value)),
            version=version)
```
> TODO: Describe label formatter.
```python
class CollectionLabelFormatter():
    pass
```


Once we read the `BratRelation` instances, in further there is a need to perform a conversion to `TextOpinion`.
TextOpinion is a general type in AREkit framework which describes a connection between a pair of objects mentioned in text:
```python
class CollectionOpinionConverter(object):

    @staticmethod
    def to_text_opinion(brat_relation, doc_id, label_formatter):
        return TextOpinion(doc_id=doc_id,
                           text_opinion_id=int(brat_relation.ID),
                           source_id=brat_relation.SourceID,
                           target_id=brat_relation.TargetID,
                           label=label_formatter.str_to_label(brat_relation.Type))
```

Now we set everything up in order to finally declare our reader.
The snippet below provides its implementation:

```python
class CollectionDocReader(object):

    @staticmethod
    def read_text_opinions(filename, doc_id, version, label_formatter):
        return CollectionIOUtils.read_from_zip(
            inner_path=CollectionIOUtils.get_annotation_innerpath(filename),
            process_func=lambda input_file: [
                CollectionOpinionConverter.to_text_opinion(relation, doc_id, label_formatter)
                for relation in BratAnnotationParser.parse_annotations(input_file)["relations"]
                if label_formatter.supports_value(relation.Type)],
            version=version)

    @staticmethod
    def read_document(filename, doc_id, label_formatter, version=CollectionVersions.V1):
        
        def file_to_doc(input_file):
            sentences = BratDocumentSentencesReader.from_file(input_file, entities)
            return BratNews(doc_id, sentences, text_opinions)
            
        entities = CollectionEntityCollection.read_collection(filename, version)
        text_opinions = CollectionNewsReader.read_text_opinions(
            filename, doc_id, version, label_formatter)
        return CollectionIOUtils.read_from_zip(
            inner_path=CollectionIOUtils.get_news_innerpath(filename),
            process_func=file_to_doc,
            version=version)
```

Every document is considered to be a list of sentences, where every sentence is an ordinary text. Lets put some details on how we perform reading ...

### Testing Implemented Collection Reader

Now we can adopt the collection reader in order to receive an instances with annotated named entities (`BratNews`).
We first provide the information about every sentences, i.e. the related text and list of mentioned named entities in it.
Then, using `TextOpinion` property, we display list of all the relations presented in annotation file.

```python

news = CollectionDocReader.read_document("0", doc_id=0, label_formatter=CollectionLabelFormatter())
assert(isinstance(news, BratNews))
for sentence in news.iter_sentences():
    assert(isinstance(sentence, BratSentence))
    print(sentence.Text.strip())
    for entity, bound in sentence.iter_entity_with_local_bounds():
        print("{}: ['{}',{}, {}]".format(
            entity.ID, entity.Value, entity.Type, 
            "-".join([str(bound.Position), str(bound.Position+bound.Length)])))

    print()

for text_opinion in news.TextOpinions:
    assert(isinstance(text_opinion, TextOpinion))
    print(text_opinion.SourceId, text_opinion.TargetId, str(type(text_opinion.Sentiment)))
```

The example output is as follows:
```
...
Президент США Джордж Буш заявил, что продолжение мирных переговоров на Ближнем 
востоке возможно только в том случае, если ХАМАС разоружится и признает государство Израиль.
 Такого же мнения придерживаются большинство европейских экспертов.
31: ['президент',PROFESSION, 0-9]
71: ['сша',COUNTRY, 10-13]
32: ['джордж буш',PERSON, 14-24]
34: ['ближнем востоке',LOCATION, 71-86]
35: ['хамас',ORGANIZATION, 122-127]
36: ['израиль',COUNTRY, 163-170]
37: ['европейских',LOCATION, 216-227]
...
...
...
1 2 <class 'labels.types.PositiveTo'>
5 4 <class 'labels.types.PositiveTo'>
21 20 <class 'labels.types.PositiveTo'>
24 25 <class 'labels.types.NegativeTo'>
29 30 <class 'labels.types.NegativeTo'>
```

-------------------------------------
NEXT POSTs

The collection binding denotes that we may declare API for our custom dataset in order to provide `ParsedNews`.
Parsed news represent a tokenized version of the document contents, i.e. the in a form of a sequence of the separated terms where every term might relate to any type. The common types are: words or entities.