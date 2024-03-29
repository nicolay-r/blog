---
layout: post
title: "AREkit Tutorial: Data Folding Setup"
description: "AREkit Tutorial: Data Folding Setup"
category: POST
tags: [AREkit, CV, Split, Folding]
---

![alt text](https://raw.githubusercontent.com/nicolay-r/blog/master/img/arekit-data-folding.png)

Besides the text contents processing, retrieving information and annotating inner objects in texts, 
in case of Machine Learning models it is also required to manage data subsets.
In other words, there is a need to provide rules on how the series of documents is expected to be grouped in *subsets*.
For example, we may declare subsets for: **Training**, **Testing**, **Validating**, etc.
This post represents an AREkit tutorial devoted to `Folding` as a common type and the related concept on how data 
separation might be described and passed into other pipelines required so.

<!--more-->

> [Tutorial code](https://github.com/nicolay-r/AREkit/blob/master/tests/tutorials/test_tutorial_data_foldings.py)

In order to describe document separation format, AREkit-0.22.1 provides the `BaseDataFolding` type,
which in short could be described in the following snippet:
```python
class BaseDataFolding(object):
    # ... other contents
    def fold_doc_ids_set(self):
        raise NotImplementedError()
```

According to the implementation above, declaring your own folding required `fold_doc_ids_set` implemenation.
This implementation considers that for a given set of data types, such as `Train`, `Test`, `Dev`, and many others,
we declare a set of the related documents.

## Folding Types

One of the common type of folidings is the predefined one, or **fixed**. 
In this type we consider a predefined separation of the document indices between predefined data types.
This folding type might be initialized as follows:
```python
parts = {
    DataType.Train: [0, 1, 2, 3],
    DataType.Test: [4, 5, 6, 7]
}
fixed_folding = FixedFolding.from_parts(parts)
```

The absence of folding at all could be declared as follows:
```python
no_folding = NoFolding(doc_ids=[10, 15, 20], supported_data_type=DataType.Dev)
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

As for `doc_ops` parameter, this parameter is related to `DocumentOperation` type, which we cover in the
[AREkit Tutorial: Craft your text-opinion annotation pipeline!](https://github.com/nicolay-r/blog/blob/master/_posts/2022-08-31-arekit-text-opinion-annotation-pipeline.md).
```python
doc_ops = FooDocumentOperations()
splitter_statistical = StatBasedCrossValidationSplitter(
    docs_stat=SentenceBasedDocumentStatGenerator(lambda doc_id: doc_ops.get_doc(doc_id)),
    docs_stat_filepath_func=lambda: "data/stat.txt")
```

Once any of the splitter types declared, it is possible to initialize Cross-Validational-based folding.
AREkit provides a two class folding type, which could be adopted as follows:
```python
cv_folding = TwoClassCVFolding(supported_data_types=[DataType.Train, DataType.Test],
                               doc_ids_to_fold=list(range(10)),
                               cv_count=3,
                               splitter=splitter_simple)
```

## Display Data-Type Foldings

For all the folding types mentioned above, AREkit provides `fold_to_doc_ids_set` method which returns dictionary of foldings. 
The snippet below illustrates on how the folding could be obtained:
```python
def show_folding(folding):
    assert(isinstance(folding, BaseDataFolding))
    split_dict = folding.fold_doc_ids_set()
    for data_type, doc_ids in split_dict.items():
        print(data_type, doc_ids)
```

In case of the *k-fold Cross Validation* there is an additional `iter_states` method which allows us
to iterate over a different `k` states. 
Combining this call with a fuction declared above, all the possible data type foldings might be obtained as presented below:
```python
for state_index, _ in enumerate(cv_folding.iter_states()):
    print("State: ", state_index)
    show_folding(cv_folding)         
```

## Conclusion

> [Tutorial code](https://github.com/nicolay-r/AREkit/blob/master/tests/tutorials/test_tutorial_data_foldings.py)

Thank you for reading this post! Now you're able to choose and declare your folding in order to then pass it for samples generation. 
However, the latter is a part of another topic. 
Stay well and connected for updates.
