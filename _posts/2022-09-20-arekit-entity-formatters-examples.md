---
layout: post
title: "AREkit Tutorial: Entity Values Formatting Examples"
description: "AREkit Tutorial: Entity Values Formatting Examples"
category: POST
visible: 0
tags: [AREkit, Entity, Masking, Examples]
---

```python
class OpinionEntityType(Enum):
    Object = 1
    Subject = 2
    SynonymSubject = 3
    SynonymObject = 4
    Other = 5
```

```python
class StringEntitiesFormatter(object):
    def to_string(self, original_value, entity_type):
        raise NotImplementedError()
```

```python
class CustomEntitiesFormatter(StringEntitiesFormatter):

    def __init__(self, subject_fmt="[subject]", object_fmt="[object]"):
        self.__subj_fmt = subject_fmt
        self.__obj_fmt = object_fmt

    def to_string(self, original_value, entity_type):
        if entity_type == OpinionEntityType.Other:
            return original_value
        elif entity_type == OpinionEntityType.Object or
                entity_type == OpinionEntityType.SynonymObject:
            return "[object]"
        elif entity_type == OpinionEntityType.Subject or
                entity_type == OpinionEntityType.SynonymSubject:
            return "[subject]"
        return None
```
