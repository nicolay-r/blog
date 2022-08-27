---
layout: post
title: "AREkit Entity Formatters"
description: "AREkit Entity Formatters"
category: POST
visible: 0
tags: [AREkit, Entity, Masking]
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
