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

Lets get started ... Considering that we have a collection `foo.zip` 
with the following structure, presented in a form of the archive:

```
foo.zip/
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
class FooVersions(Enum):
    V1 = "V1"
```

Next, there is a need to establish connection between files and its contents.
Source-related contribution part provides API for interaction with contents in `zip` archives (class `ZipArchiveUtils`). 
To adopt the latter we provide an inherited class by declaring the following information: 
* archive filepath
* news and annotation filenames

```python
class FooIOUtils(ZipArchiveUtils):

    archive_path = "foo.zip"

    @staticmethod
    def get_archive_filepath(version):
        return FooIOUtils.archive_path

    @staticmethod
    def get_annotation_innerpath(filename):
        return "{}.ann".format(filename)

    @staticmethod
    def get_news_innerpath(filename):
        return "{}.txt".format(filename)

    @staticmethod
    def __iter_filenames_from_dataset():
        for filename in FooIOUtils.iter_filenames_from_zip(FooVersions.V1):
            yield basename(filename)

    @staticmethod
    def iter_collection_filenames():
        filenames_it = FooIOUtils.__iter_filenames_from_dataset()
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
class FooEntityCollection(EntityCollection):

    def __init__(self, contents, value_to_group_id_func):
        super(FooEntityCollection, self).__init__(contents["entities"], value_to_group_id_func)
        self._sort_entities(key=lambda entity: entity.IndexBegin)

    @classmethod
    def read_collection(cls, filename, version):
        synonyms = StemmerBasedSynonymCollection(
            iter_group_values_lists=[], stemmer=MystemWrapper(), is_read_only=False, debug=False)
        return FooIOUtils.read_from_zip(
            inner_path=FooIOUtils.get_annotation_innerpath(filename),
            process_func=lambda input_file: cls(
                contents=BratAnnotationParser.parse_annotations(input_file),
                value_to_group_id_func=lambda value:
                    SynonymsCollectionValuesGroupingProviders.provide_existed_or_register_missed_value(
                        synonyms, value)),
            version=version)
```

Now we set everything up in order to finally declare our reader.
The snippet below provides its implementation:

```python
class FooDocReader(object):

    @staticmethod
    def read_text_relations(filename, version):
        return FooIOUtils.read_from_zip(
            inner_path=FooIOUtils.get_annotation_innerpath(filename),
            process_func=lambda input_file: [relation for relation in 
                BratAnnotationParser.parse_annotations(input_file)["relations"]],
            version=version)

    @staticmethod
    def read_document(filename, doc_id, version=CollectionVersions.V1):
        
        def file_to_doc(input_file):
            sentences = BratDocumentSentencesReader.from_file(input_file, entities)
            return BratNews(doc_id, sentences, text_relations)
            
        entities = FooEntityCollection.read_collection(filename, version)
        text_relations = CollectionNewsReader.read_text_relations(filename, version)

        return FooIOUtils.read_from_zip(
            inner_path=FooIOUtils.get_news_innerpath(filename),
            process_func=file_to_doc,
            version=version)
```

Every document is considered to be a list of sentences, where every sentence is an ordinary text. Lets put some details on how we perform reading ...

### Testing Implemented Collection Reader

Now we can adopt the collection reader in order to receive an instances with annotated named entities (`BratNews`).
We first provide the information about every sentences, i.e. the related text and list of mentioned named entities in it.
Then, using `Relations property, we display list of all the relations presented in annotation file.

```python
news = FooDocReader.read_document("0", doc_id=0)
for sentence in news.iter_sentences():
    print(sentence.Text.strip())
    for entity, bound in sentence.iter_entity_with_local_bounds():
        print("{}: ['{}',{}, {}]".format(
            entity.ID, entity.Value, entity.Type, 
            "-".join([str(bound.Position), str(bound.Position+bound.Length)])))

for brat_relation in news.Relations:
    print(brat_relation.SourceID, brat_relation.TargetID, brat_relation.Type)
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
1 2 PositiveTo
5 4 PositiveTo
21 20 PositiveTo
24 25 NegativeTo
29 30 NegativeTo
```

Finally we have a `BratNews` instance which contains information about mentioned named entities
and relations. In next posts we provide details on cases when this type of news might be utilized, 
for such cases as
 * text parsing, 
 * data sampling, and so on. 

Thank you for reading this post!
