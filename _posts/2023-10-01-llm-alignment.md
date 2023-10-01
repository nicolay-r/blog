---
layout: post
title: "The Recent Advances in LLM Alignment With Manually Annotated Relations in Mass-Media News"
description: "The Recent Advances in LLM Alignment With Manually Annotated Relations in Mass-Media News"
category: POST
tags: [AREkit, arekit-ss, ChatGPT-3.5, GPT, LLM, ChatGPT, relations, NEREL, dataset, analysis, reasoning, sampling]
---

![alt text](https://raw.githubusercontent.com/nicolay-r/blog/master/img/2023-10-01-llm-alignment-logo.png)

Let me share a quick summary and update on misalignment between LLM reasonining and manually annotated relations in mass media-news.

Almost a month ago we discovery the recent advanced in LLM reasoning. All the discussed and analyzed materials were formed into presentation with following slides (in Russian):

✨ 📰 Slides: https://nicolay-r.github.io/website/data/report_llm2023-nerel.pdf 

Here is the three keypoint you might take out of it:

💚 1. **High level of agreement.** Reasoning within context for time-based relations. They are very grammar dependent, which it becomes the may reason model has a huge alignment rate. (Credits to transformer architecture and self-attention)

💛 2. **Medum level of agreement.** Locations and position related types of relations. LLM models may end up with non-definitive meaning of relation types. The exceptional cases are: relations that involve well-known politicians and countries.

❤️ 3. **Low level of agreement** were mainly caused by (i) terminology misalignment and (ii) sharper reasoning from LLM. This is a case where context is expected to be enriched and/or the meaning of relation type is better described.

For everything mentioned above, we eleminate such cases as: (i) issues with translation, (ii) network hallucination. If the first was mostly a technical trait of implementation, the hallucination is one thing that still remains poorly covered.

And thanks for AREkit double-s (AREkit-ss) which makes these studies available to replicate and spread on other languages. Feel free to check out or share them below to sample data with the one line! 🔥

💻 https://github.com/nicolay-r/arekit-ss/issues/52

Thank you for reading, interest and support! 🙏

🌟 AREkit "double-s": https://github.com/nicolay-r/arekit-ss 

🌟 AREkit core: https://github.com/nicolay-r/AREkit 