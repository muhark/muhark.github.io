---
title: "IssueBench: Millions of Realistic Prompts for Measuring Issue Bias in LLM Writing Assistance"
authors: "Paul Röttger, <b>Musashi Hinck</b>, Valentin Hofmann, Kobi Hackenburg, Valentina Pyatkin, Faeze Brahman, Dirk Hovy"
collection: publications
category: preprints
permalink: /preprints/issuebench
excerpt: ''
paperurl: 'https://arxiv.org/abs/2502.08395'
---

- [Dataset](https://huggingface.co/datasets/Paul/IssueBench)
- [Repo](https://github.com/paul-rottger/issuebench)
- [Blogpost (LinkedIn)](https://www.linkedin.com/pulse/issuebench-millions-realistic-prompts-measuring-issuebias-hinck-o379e/)



## Abstract

Large language models (LLMs) are helping millions of users write texts about diverse issues, and in doing so expose users to different ideas and perspectives. This creates concerns about issue bias, where an LLM tends to present just one perspective on a given issue, which in turn may influence how users think about this issue. So far, it has not been possible to measure which issue biases LLMs actually manifest in real user interactions, making it difficult to address the risks from biased LLMs. Therefore, we create IssueBench: a set of 2.49m realistic prompts for measuring issue bias in LLM writing assistance, which we construct based on 3.9k templates (e.g. "write a blog about") and 212 political issues (e.g. "AI regulation") from real user interactions. Using IssueBench, we show that issue biases are common and persistent in state-of-the-art LLMs. We also show that biases are remarkably similar across models, and that all models align more with US Democrat than Republican voter opinion on a subset of issues. IssueBench can easily be adapted to include other issues, templates, or tasks. By enabling robust and realistic measurement, we hope that IssueBench can bring a new quality of evidence to ongoing discussions about LLM biases and how to address them. 
