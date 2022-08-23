Examples of the text processings.

In [AREkit-0.22.1](https://github.com/nicolay-r/AREkit), 
we provide `BaseTextParser` which assumes to apply a series of text processing items, organized in a form of the `PipelineItem`'s,
in order to modify the text contents with the annotated objects in it. 
The common class which is related to performing transformation from the original `News` towards the processed one is a `BaseTextParser`, which receives
a pipeline of the **annotations** expected to be applied towards a news text:

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

### Entities Annotation

Among all of the potential annotations that could be applied towards text, 
the most important one is related to the **annotation of the mentioned named entities** (NE) in text.
This task might be performed manually and provided as part of the document annotation itself, or 
the latter might be accomplished automatically. 
We may encounter with the predefined annotation once we treating the BRAT-based collections, 
or other collections that provides such annotation by default.

```python
# For the already known Named-Entities, provided by the BRAT-based collection.
text_parser = BaseTextParser([BratTextEntitiesParser()])
```

In case of a raw texts we deal with the NER task, dubbed as *Named Entity Recognition* problem.
> **NOTE:** We cover this scenario in a greater deails as a part of the 
[ARElight](https://github.com/nicolay-r/ARElight/blob/v0.22.1/arelight/text/ner_ontonotes.py) project, 
by adopting BERT models for NER via DeepPavlov framework (`BertOntonotesNER` class); 
we left so outside of the following post and kindly refer you to the related project once there is a need 
in examine so in a greater details

The snippet below illustrates on how the `BERT_ontonotes` model could be adopted for an automatic 
named entities annotation:

```python
text_parser = BaseTextParser([
    BertOntonotesNERPipelineItem(lambda s_obj: s_obj.ObjectType in ["ORG", "PERSON", "LOC", "GPE"])
])
```
### Terms annotation

Besides the mentioned named entities itself, mostly there is a need to separate words from each other.
We treat this operation in this post as a *tokenization* process. 
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

### Frames annotation
```python
stemmer = MystemWrapper()

frames_collection = RuSentiFramesCollection.read_collection(
    version=RuSentiFramesVersions.V20,
    labels_fmt=RuSentiFramesLabelsFormatter(pos_label_type=PositiveTo, neg_label_type=NegativeTo),
    effect_labels_fmt=RuSentiFramesEffectLabelsFormatter(pos_label_type=PositiveTo, neg_label_type=NegativeTo))
    frame_variant_collection = FrameVariantsCollection()
    frame_variant_collection.fill_from_iterable(
        variants_with_id=frames_collection.iter_frame_id_and_variants(),
        overwrite_existed_variant=True,
        raise_error_on_existed_variant=False))

text_parser = BaseTextParser(pipeline=[
    BratTextEntitiesParser(),
    DefaultTextTokenizer(keep_tokens=True),
    FrameVariantsParser(frame_variant_collection)
])
```

