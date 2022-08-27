---
layout: post
title: "AREkit Tutorial: Inferred Results Evaluation!"
description: "AREkit Tutorial: Inferred Results Evaluation!"
category: POST
tags: [AREkit, Evaluation, Evaluator, Precision, Recall, Accuracy]
---

Once the relation extraction model is ready-to-use, it is expected to be evaluated and compared with the other approaches. 
Given a couple datasets of a `Test` and `Etalon` types, it is possible to assess the result model using the 
AREkit embedded evaluators.

In this posts we cover a possible evaluation formats for Sentiment Relations of the Relation Extraction subtask.

<!--more-->

> NOTE: The present **AREkit-0.22.1** has a limitation onto 2 or 3 class problems for evaluation 
[#363](https://github.com/nicolay-r/AREkit/issues/363).
We looking forward to address on this issue in further. 

## Required Data

According to the prior tutorials, i.e.
folding and 
sampling, 
for a given set of documents or binded collections, using AREkit API, 
it is possible to declare and infer samples splitted onto one of the following **samples sets**:
1. `TEST` -- annotation (usually) with the unknown label (`NoLabel` instance in AREkit terminology)
2. `ETALON` -- annotation with the known information and known labels.

The reason is why we provide a custom measurement implementations is as follows. 
Performing the relation extraction we can not guarantee the equalty of the `TEST` and `ETALON` parts due to the:
* Adoptation of the different text-opinion extraction pipelines (common case)
* Affection of the coreference resolutions (in case when we don't know all the synonymy groups in advance).

Once we apply our pre-trained model towards the `TEST` data, we may receive:

3. `Predict-TEST` -- annotated data of the related `TEST` set;

Важный момент, что в TEST данных больше, так как мы решаем задачу извлечения (extraction), поэтому нам нужно еще выделить контексты!!!

## Evaluation Formats

И затем, как имея такую информацию можно оценить результат:

На уровне документа (тогда есть агрегация и прочее, можно на свою статью сослаться, взять код метода из нивц).
На уровне отедельных отношений в рамках текста.
