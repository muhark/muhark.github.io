---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

# Education

## DPhil University of Oxford, 2022

_Supervisors_: Andrew C. Eggers, Raymond Duch

Wrote doctoral thesis on applications of RL and NLP in political campaigns:

1. Conducted human subjects research demonstrating that offline RL could be used to improve the effect of targeted political advertisements.
2. Designed and trained multimodal transformer-based encoder for generating image-and-text embeddings from political Facebook ads.
3. Compared structured and unstructured NLP tools for document retrieval of economic narratives in newspaper corpora.

Teaching: Python, statistics and HPCs to social scientists.

## MSc University of Oxford, 2019

_Supervisor_: Andrew C. Eggers

Focused on quantitative/statistical methods for causal inference and NLP.

## BA University of Oxford, 2015

Read Philosophy, Politics and Economics at Merton College, Oxford.



# Employment

## **AI Research Scientist**, Intel Labs, USA (2024-present)

Staff scientist with Multimodal Cognitive AI group under PI Vasudev Lal.

Research Highlights:

- First author on two new projects in first five months.
- Received Outstanding Paper Award at ACL 2024 for work on measuring latent values and opinions in LLMs.
- Won the Scholar Award for highest academic output in first year at Intel Labs.

Engineering Highlights:

- Ported cutting-edge multimodal models to new architectures.
- First team to execute foundation model training at 1K Gaudi 2 card scale in Intel Tiber AI Cloud.

## **Postdoctoral Research Associate**, Princeton University, NJ, USA (2022-2024)

Postdoctoral researcher at [Data Driven Social Science Initiative](https://ddss.princeton.edu) working with [Professor Brandon Stewart](https://bstewart.scholar.princeton.edu).

- Published [statistical framework for using LLMs for data annotation without introducing bias](/publication/dsl-neurips-2023).
- Gave workshops on [LLMs](https://github.com/muhark/nn-tutorial/blob/main/part2/lecture.md) and [intro deep learning](https://github.com/muhark/nn-tutorial/blob/main/part1/lecture.md) to generalist audiences.
- Developed LLM agent-based tool for improving survey research.


## **Predoctoral Research Fellow**, University College London, London, UK (2021-2022)

Predoctoral researcher on UKRI-funded MENMOPE project led by Professor Lucy Barnes.

## **Quantitative Analyst**, SBI Group, Tokyo, Japan (2016-2018)

Data scientist/analyst at SBI Japannext, Japan's foremost PTS.

- Automated reporting for stock market clients from terabyte-scale trading engine data.
- Used ML/econometrics to provide business intelligence for internal and external stakeholders.


# Skills

Experienced with following:

- Languages: `Python`, `R`, `bash`, `SQL`
- Model development with `pytorch`, `transformers`
- Distributed training with `torch.distributed`, `deepspeed`, `FSDP`
- Experienced with `docker`, `kubernetes` and `SLURM`



# Publications

<ul>
{% for post in site.publications reversed %}
    {% include archive-single.html %}
{% endfor %}
</ul>
  
<!-- Talks
======
  <ul>{% for post in site.talks reversed %}
    {% include archive-single-talk-cv.html  %}
  {% endfor %}</ul> -->
  
<!-- Teaching
======
  <ul>{% for post in site.teaching reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
  
-->