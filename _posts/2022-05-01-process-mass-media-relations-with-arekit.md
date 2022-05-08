---
layout: post
title: "Process Mass-Media relations for Language Models with AREkit" 
description: "Process Mass-Media relations for Language Models with AREkit"
category: POST
tags: [Relation Extraction, AREkit, Language Models, BERT, tutorials]
comments: true
---

![alt text]({{site.url}}/img/arekit_for_mass_media.png)

Automatic processing of a large documents requires a deep text understanding,
including connections between objects extraction.
In terms of the latter and such domain as **Sentiment Analysis**, capturing the 
sentiments between objects (relations) gives a potential for further analysis, 
built of top of the sentiment connections: 

<!--more-->

![alt text](https://github.com/nicolay-r/ARElight/blob/main/docs/inference-bert-e1.png?raw=true)

In this post we're focusing on the Sentiment Relation Extraction between mentioned 
entities in texts, written in Russian.
 This analysis finds its application in analytical texts, in which
author sharing his opinion onto variety of objects. The latter causes some text objects 
to become a source of opinions in texts.
More onto the related task could be found in 
[this paper](https://arxiv.org/pdf/1808.08932.pdf) or at 
[NLP-progress](http://nlpprogress.com/russian/sentiment-analysis.html).

For this Sentiment Relation Extraction we adopt and design [AREkit framework (0.22.0)](https://github.com/nicolay-r/AREkit/tree/0.22.0-rc).
Let's take a closer look on how this framework (set of toolkits) allows us to prepare the information 
in order to initiate automatic relation extraction by means of machine learning methods.
In particular, within this post we are focused on [BERT language models](https://arxiv.org/pdf/1810.04805.pdf).

All the snippets below organized in a example (unit-test script) withing a side project dubbed as 
[ARElight](https://github.com/nicolay-r/ARElight) which may be found 
[here](https://github.com/nicolay-r/ARElight/blob/main/test/test_bert_serialization.py).

> [LINK TO A COMPLETE EXAMPLE](https://github.com/nicolay-r/ARElight/blob/main/test/test_bert_serialization.py)

Here, we start by declaring a list of texts it is expected to be processed. 
In particular we consider a list of a single document:
```python
texts = [
    # Text 1.
    """24 марта президент США Джо Байден провел переговоры с
       лидерами стран Евросоюза в Брюсселе, вызвав внимание рынка и предположения о
       том, что Америке удалось уговорить ЕС совместно бойкотировать российские нефть
       и газ.  Европейский Союз крайне зависим от России в плане поставок нефти и
       газа."""  
]
```

Next, we declare a **synonyms collection**.
It is provided by AREkit and allows us to address on the named entity co-reference problem.
In order to initialize this collection, there is a need to provide a list of synonym groups 
([see an example](https://raw.githubusercontent.com/nicolay-r/ARElight/main/data/synonyms.txt)).
Since the entity values are in russian, it expected to represent them in a lemmatized format.
For the latter we adopt a wrapper over [mystem](https://github.com/aotd1/mystem) (`MystemWrapper`) library.
While using synonyms collection, it is expected that in some cases entries might not be found 
and therefore we allow synonyms collection expansion by treating it in a **non read-only mode** 
(`is_read_only` flag is `False`).

```python
synonyms_filepath = "synonyms.txt"

def iter_groups(filepath):
    with open(filepath, 'r', encoding='utf-8') as file:
        for data in iter_synonym_groups(file):
            yield data

synonyms = StemmerBasedSynonymCollection(
    iter_group_values_lists=iter_groups(synonyms_filepath),
    stemmer=MystemWrapper(),
    is_read_only=False,
    debug=False)
``` 

On the second step, we implement **text parser**. 
In AREkit-0.22.0, the latter represents a list of the transformations organized in a pipeline 
of the following transformations:
1. We split the input sequence onto list of words separated by white spaces.
2. For named entities annotation we adopt BERT model (pretrained on the OntoNotes-v5 collection), provided by 
[DeepPavlov](https://deeppavlov.ai/).
3. Since we perform entities grouping, there is a need to provide the related function (`get_synonysms_group_index`):

```python
def get_synonym_group_index(s, value):
    if not s.contains_synonym_value(value):
        s.add_synonym_value(value)
    return s.get_synonym_group_index(value)

text_parser = BaseTextParser(pipeline=[
    TermsSplitterParser(),
    BertOntonotesNERPipelineItem(lambda s_obj: s_obj.ObjectType in ["ORG", "PERSON", "LOC", "GPE"])
    EntitiesGroupingPipelineItem(lambda value: get_synonym_group_index(synonyms, value))
])
```

Next, we declare the **opinion annotation algorithm** required to compose a PAIRS of objects 
across all the objects mentioned in text.
All the pairs are considered to be annotated with the unknown label (`NoLabel`).
In addition, it is possible to clarify the max distance bound in terms between pair participants, or leave it `None` 
by considering a pair without this parameter:
```python
algo = PairBasedAnnotationAlgorithm(
    label_provider=ConstantLabelProvider(label_instance=NoLabel()),
    dist_in_terms_bound=None)

annotator = DefaultAnnotator(algo)
```

Then, we focusing on experiment context implementation.

* Declare a **folding** over list of documents.
In general, you may organize a split of document onto `Train` and `Test` sets.
Within this example, we
adopt `NoFolding` and consider that all the documents will be witin a single (`DataType.TEST`) list.
* Declare **labels formatter**, which is required for `TextB` of the BERT input sequence in further.
* Declare **entities formatter**, considering masking them by providing a BERT styled, sharp-prefixed tokens:
 `#O` (Object) and `#S` (Subject).

```python
no_folding = NoFolding(doc_ids_to_fold=list(range(len(texts))),
                       supported_data_types=[DataType.Test])

labels_fmt = StringLabelsFormatter(stol={"neu": NoLabel})

str_entity_formatter = SharpPrefixedEntitiesSimpleFormatter()
```

On last prepartion step, we gather everyting together in order to compose an **experiment handler**.

Being a [core module of AREkit](https://github.com/nicolay-r/AREkit/tree/0.22.0-rc/arekit/common/experiment), 
**experiment** -- is a sets of API components required to work with a large amount of mass-media 
news. This set includes and consider initialization of the following components:
1. context  -- required for sampling.
2. input/output -- methods related to I/O operations organization
3. document related operations;
4. opinion related operations;
5. List of handlers.

For context initialization:
* Contexts with a mentioned subject object pair in it, limited by `50` terms;
* `NoLabel()` instance to label every sample.
```python
exp_ctx = BertSerializationContext(
    label_scaler=SingleLabelScaler(NoLabel()),
    annotator=annotator,
    terms_per_context=50,
    str_entity_formatter=str_entity_formatter,
    name_provider=ExperimentNameProvider(name="example-bert", suffix="serialize"),
    data_folding=no_folding)

exp_io = InferIOUtils(exp_ctx=exp_ctx, output_dir="out")

doc_ops = CustomDocOperations(exp_ctx=exp_ctx, text_parser=text_parser)

exp = CustomExperiment(
    synonyms=synonyms,
    exp_io=exp_io,
    exp_ctx=exp_ctx,
    doc_ops=doc_ops,
    labels_formatter=labels_fmt,
    neutral_labels_fmt=labels_fmt)
```

In terms of experiment handler, which is related to data preparation for BERT model, additionally declaring:
* `nli-m` text formatter for `TextB` and utilize NLI approach by empasizing the context
  between Object and Subject pair ([see original paper](https://arxiv.org/pdf/1903.09588.pdf)).
```handler
handler = BertExperimentInputSerializerIterationHandler(
    exp_io=exp_io,
    exp_ctx=exp_ctx,
    doc_ops=doc_ops,
    opin_ops=exp.OpinionOperations,
    sample_labels_fmt=labels_fmt,
    annot_labels_fmt=labels_fmt,
    sample_provider_type=BertSampleProviderTypes.NLI_M,
    entity_formatter=exp_ctx.StringEntityFormatter,
    value_to_group_id_func=synonyms.get_synonym_group_index,
    balance_train_samples=True)     # This parameter required by not utilized.
```

> **NOTE:** In further versions we're looking forward to adopt dictionaries to pass these parameters, [see issue #318](https://github.com/nicolay-r/AREkit/issues/318)

The application of the handler towards the provided lists of texts is based on the `ExperimentEngine`.
The engine allows to execute handler by passing a folding schema.
 we adopt experiment `engine`:
```python
def input_to_docs(texts):
    docs = []
    for doc_id, contents in enumerate(texts):
        sentences = ru_sent_tokenize(contents)
        sentences = list(map(lambda text: BaseNewsSentence(text), sentences))
        doc = News(doc_id=doc_id, sentences=sentences)
        docs.append(doc)
    return docs

docs = input_to_docs(texts)
doc_ops.set_docs(docs)
engine = ExperimentEngine(exp_ctx.DataFolding)  # Present folding limitation.
engine.run([handler])
```

> **NOTE:** We demonstrate it in a simple way, consuming `O(N)` memory (`N`-amount of documents). However it is possible to perform document reading on demand, to 
> prevent keep all the document in memory.

Finally, the output represents a multiple files, including opinions, **samples**:
```
out/
   opinion-test-0.tsv.gz
   sample-test-0.tsv.gz     <-- Required.
   result_d0_Test.txt
```

> **NOTE:** Other files generated after handler application are not utlized within this example and might be 
>removed in future AREkit releases. [see #282](https://github.com/nicolay-r/AREkit/issues/282)

For **samples**, every record represent a context sample with the corresponding features:
```csv
id	doc_id	label	text_a	text_b	s_ind	t_ind	sent_ind	entity_values	entity_types	
o0_i0_	0	0	24 марта президент #E #S провел переговоры с лидерами стран #O в #E вызвав вни ...
o0_i1_	0	0	24 марта президент #E #S провел переговоры с лидерами стран #O в #E вызвав вни ...
...
```

Thank you for your attention!

## References

[1]. [AREkit -- is an Attitude and Relation Extraction toolkit](https://github.com/nicolay-r/AREkit)

[2]. [BERT: Pre-training of Deep Bidirectional Transformers for
Language Understanding](https://arxiv.org/pdf/1810.04805.pdf)

[3]. [Utilizing BERT for Aspect-Based Sentiment Analysis
via Constructing Auxiliary Sentence](https://arxiv.org/pdf/1903.09588.pdf)
