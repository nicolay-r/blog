---
layout: post
title: "AREkit and What's new in Release 0.22.1"
description: "AREkit and What's new in Release 0.22.1"
category: POST
tags: [AREkit, Relese, AREkit-0.22.1, Pipelines, BRAT, OpenNRE, Tutorials, SeqIO]
---

![alt text](https://user-images.githubusercontent.com/14871187/188810264-d7ea509b-6d6b-4cd4-bebd-cf1f15f9d4a9.png)

We are happy to announce the [AREkit version 21.1](https://github.com/nicolay-r/AREkit).
 
The complete and large [list of the updates](https://github.com/nicolay-r/AREkit/releases/tag/v0.22.1-rc) 
you can find on the release page in greater details.

<!-- more -->

In short we first provide greater support of the BRAT-based annotation in order
to adopt it in your own collections in order to use them for attitudes
annotation and document sampling for training your machine learning.
We proceed to develop a toolset and treat it as a preprocessor for [OpenNRE framework](https://github.com/thunlp/OpenNRE). 
OpenNRE allows you to quickly develop a model adopted for the relation extraction specific problem, by being based on the JSON-like
prepared data in terms of training/fine-tunning/evaluation stages. 
For now, AREkit allows us to do the preprocessing work with 
[thespecific writer](https://github.com/nicolay-r/AREkit/blob/0.22.1-rc/arekit/common/data/input/writers/opennre_json.py).

Alongside with the AREkit, we also update the [ARElight-21.1](https://github.com/nicolay-r/ARElight). 
The latter allows you to perform the quick stabilization of the
input text from your Council.And you can also find the update.
Please follow the release notes for a greater details according to the [following link](https://github.com/nicolay-r/ARElight/releases/tag/0.22.1-p1).

![alt text](https://user-images.githubusercontent.com/14871187/188832503-1bb27da4-97cf-48d7-ae52-026e75c38721.png)

And finally we have a [nice tutorials journey](https://github.com/nicolay-r/AREkit/tree/master/tests/tutorials)! (see image below)
This is a list of examples over
the sequence of the three major points of the application. We collect all the
tutorials and separate them [onto the three stage](https://nicolay-r.github.io/blog/articles/2022-08/arekit-sources-sampling-pipeline), related to
1. such as embedding the custom
collection.
2. app creation of the pipelines for [**processing texts**](https://nicolay-r.github.io/blog/articles/2022-08/arekit-text-parsing-pipeline), 
for a way of [**annotating attitudes**](https://nicolay-r.github.io/blog/articles/2022-08/arekit-text-opinion-annotation-pipeline) in it. 
3. contexts with text opinions [serialization](https://nicolay-r.github.io/blog/articles/2022-09/arekit-sampling-bert).

![alt text](https://raw.githubusercontent.com/nicolay-r/blog/master/img/arekit-sources-sampling-pipeline.png)

And all of the mentioned above does not cover all the features added so far.
The complete and large list of the updates you can find on the release page in
greater [details](https://github.com/nicolay-r/AREkit/releases/tag/v0.22.1-rc).

Thank you for reading!
