---
layout: post
title: "AREkit Tutorial: Entity Values Formatting Examples"
description: "AREkit Tutorial: Entity Values Formatting Examples"
category: POST
visible: 0
tags: [AREkit, Entity, Masking, Examples]
---

This short post illustrates implementation of the base entity formatter.
Entity formatter is requred for formatting values of mentioned named entities, masking them.

<!--more-->

AREkit proposes an internal type `OpinionEntityType`, which describes all the possible 
types that opinion participants (subject/object) might be.
This enumeration type includes the following types:
```python
class OpinionEntityType(Enum):
    Object = 1
    Subject = 2
    SynonymSubject = 3
    SynonymObject = 4
    Other = 5
```

Implementation of the base class of the **entities formatting** illustrated in a snippet below: 
```python
class StringEntitiesFormatter(object):
    def to_string(self, original_value, entity_type):
        raise NotImplementedError()
```

Inherited versions of the base class illustrated below:
```python
class CustomEntitiesFormatter(StringEntitiesFormatter):

    def __init__(self, subject_fmt="[subject]", object_fmt="[object]"):
        self.__subj_fmt = subject_fmt
        self.__obj_fmt = object_fmt

    def to_string(self, original_value, entity_type):
        assert(isinstance(original_value, Entity))
        if entity_type == OpinionEntityType.Other:
            return original_value.Value
        elif entity_type == OpinionEntityType.Object or
                entity_type == OpinionEntityType.SynonymObject:
            return self.__obj_fmt
        elif entity_type == OpinionEntityType.Subject or
                entity_type == OpinionEntityType.SynonymSubject:
            return self.__subj_fmt 
        return None
```

List of the other entity formatters provided out-of-the-box is as follows:
* [`RussianEntitiesCasedFormatter`](https://github.com/nicolay-r/AREkit/blob/c66ea454051adfd09d37ed8a6aed143b5e2ab186/arekit/contrib/utils/entities/formatters/str_rus_cased_fmt.py#L9) -- 
supports russian cases for `subject` (`субъект`) and `object` (`объект`).

> **NOTE** It was decided not to mention other adapters since they are related to `CustomEntitiesFormatter` 
>described above.
