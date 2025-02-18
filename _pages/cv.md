---
layout: single
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}


## Employment

### **AI Research Scientist**, Intel Labs, USA (2024-present)

Staff scientist with Multimodal Cognitive AI group under PI Vasudev Lal.

#### Research Highlights

- Won the Scholar Award for highest academic output in first year at Intel Labs.
- First author on two new projects in first five months.
- Received Outstanding Paper Award at ACL 2024 for work on measuring latent values and opinions in LLMs.

#### Engineering Highlights

- Scaled foundation model training at 1K Gaudi 2 card scale in Intel Tiber AI Cloud.
- Solo developed optimized inference harness for generating >10B tokens on 128-card HPU cluster.
- Contributed to development of novel multimodal generative models.

### **Postdoctoral Research Associate**, Princeton University, NJ, USA (2022-2024)

Postdoctoral researcher at [Data Driven Social Science Initiative](https://ddss.princeton.edu) working with [Professor Brandon Stewart](https://bstewart.scholar.princeton.edu).

- Published [statistical framework for using LLMs for data annotation without introducing bias](/publication/dsl-neurips-2023).
- Gave workshops on [LLMs](https://github.com/muhark/nn-tutorial/blob/main/part2/lecture.md) and [intro deep learning](https://github.com/muhark/nn-tutorial/blob/main/part1/lecture.md) to generalist audiences.
- Developed LLM agent-based tool for improving survey research.


### **Predoctoral Research Fellow**, University College London, London, UK (2021-2022)

Predoctoral researcher on UKRI-funded MENMOPE project led by Professor Lucy Barnes.

### **Quantitative Analyst**, SBI Group, Tokyo, Japan (2016-2018)

Data scientist/analyst at SBI Japannext, Japan's largest PTS.

- Automated reporting for stock market clients from terabyte-scale trading engine data.
- Provided business intelligence for internal and external stakeholders using ML and econometric forecasting tools.

## Education

### DPhil University of Oxford, 2022

Wrote doctoral thesis on applications of RL and NLP in political campaigns:

1. Conducted human subjects research demonstrating that offline RL could be used to improve the effect of targeted political advertisements.
2. Designed and trained multimodal transformer-based encoder for generating image-and-text embeddings from political Facebook ads.
3. Compared structured and unstructured NLP tools for document retrieval of economic narratives in newspaper corpora.

Teaching: Python, statistics and HPCs to social scientists.

### MSc University of Oxford, 2019

Focused on quantitative/statistical methods for causal inference and NLP.

### BA University of Oxford, 2015

Read Philosophy, Politics and Economics at Merton College, Oxford.





## Skills

- Languages: `Python`, `R`, `bash`
- Distributed training and inference:
  - Scaled training to >1k A100-equivalents
  - Optimized inference for >10B token generation 
- ML/AIOps: `kubernetes`, `ray`, `wandb`, `SLURM` etc.
- Multimodal AI (model customization, development, training)



<!-- # Publications

<ul>
{% for post in site.publications reversed %}
    {% include archive-single.html %}
{% endfor %}
</ul> -->
  
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