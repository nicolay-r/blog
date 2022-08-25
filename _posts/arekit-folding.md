---
layout: post
title: "AREkit Tutorial: Data Folding Setup"
description: "AREkit Tutorial: Data Folding Setup"
category: POST
tags: [AREkit, CV, Split, Folding]
---

Besides the text contents processing, i.e. performing information retrieval, annotating inner objects, 
for Machine Learning models it is also required to manage data subsets.
In other words there is a need to provide rules on how the series of documents on such subsets.
For example, we may declare subset for: **Training**, **Testing**, **Validating**, etc.
In this post we propose `Folding` as a common type and the related concepts on how data separation 
might be described and the utilized in other pipelines required so.

<!--more-->

In order to describe document separation format, AREkit-0.22.1 provides the `BaseDataFolding` type,
which in short could be described in the following snippet:
```python
class BaseDataFolding(object):
    
    # ... other implementation stuff
    
    def fold_doc_ids_set(self):
        raise NotImplementedError()

```

According to the implementation above, declaring your own folding required `fold_doc_ids_set` implemenation.
This implementation considers that for a given set of data types, such as `Train`, `Test`, `Dev`, and many others,
we declare a set of the related documents.

## Folding Types

```python
data_type_foldings = {
    DataType.Train: [0, 1, 2, 3],
    DataType.Test: [4, 5, 6, 7]
}
fixed_folding = FixedFolding.from_parts(data_type_foldings)
```

The absence of folding at all could be declared as follows:
```python
no_folding = NoFolding(doc_ids=[10, 15, 20], supported_data_type=DataType.Train)
```

We also consider a combination of the different foldings by providing a `UnitedFolding` type.
For a particular data type, supported by at least one folding parameter, it gathers the related set of 
documents **and unify** of all the documents behind every provided folding. 
`UnitedFolding` type could be initialized as follows:

```python
united_folding = UnitedFolding([fixed_folding, no_folding])
```


The last folding supported by AREkit is a so-called *k-fold Cross-Validational* one.
This folding assumes to distribute whole set of documents among `k` parts.
Algorithm, which describes this distribution is a part of the so called **Splitters**.

AREkit provides two type of splitters out of the box.
The first one (simple version) is consider a random separation by a given `seed` value.
Splitter of this type could be initialized as follows:

```python
splitter_simple = SimpleCrossValidationSplitter(shuffle=True, seed=1)
```

Another type is a statistical one, in which we rely on a certain measurements, i.e. statistics
caclulated by a given document, in order to then consider this statistics for a balanced distribution.
By default we provide a sentence-based statistics generation, which calculates an amount of sentences
of a given document.
> **NOTE** in present AREkit version we consider that this statistics is provided via file (`stat.txt` according to
the snippet below)

```python
splitter_statistical = StatBasedCrossValidationSplitter(
        docs_stat=SentenceBasedDocumentStatGenerator(lambda doc_id: doc_ops.get_doc(doc_id)).
        doc_stat_filepath_func: lambda: "stat.txt"
    )
```

```python
cv_folding = TwoClassCVFolding(supported_data_types=[DataType.Train, DataType.Test],
                               doc_ids_to_fold=[1,2,3,4,5,6,7,8,9,10],
                               cv_count=3,
                               splitter=None)
```

## Conclusion

Thank you for reading this post! Now you're able to choose and declare your folding in order to then pass it for samples generation. 
However, the latter is a part of another topic. 
Stay well and connected for updates.
