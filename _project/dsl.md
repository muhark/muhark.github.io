---
layout: project_single
title: "Design-Based Supervised Learning"
slug: "dsl"
---

**Accepted to NeurIPS 2023**.

Project in collaboration with [Naoki Egami](http://naokiegami.com), [Brandon Stewart](bstewart.scholar.princeton.edu) and Hanying Wei.

We provide a framework for leveraging LLM annotations while also ensuring consistency and valid uncertainty quantification in downstream analyses.

In a few more words, with a running example:

Suppose you are interested in the use of anti-immigrant rhetoric in social media posts by politicians. Your research question (and quantity of interest) might be:

- "Are male politicians more likely to write xenophobic posts?" (effect of author trait on content)
- "Are politicians more likely to use xenophobic rhetoric in social media or TV?" (effect of medium on content)
- "Are xenophobic appeals more likely to generate engagement from other users?" (effect of content on user behavior)

The general flow would be: 1) label texts 2) run regression of feature on label (or label on feature) 3) interpret coefficients of model.
A common challenge is step 1 is prohibitively expensive (especially for large corpora). Some authors suggest that you can use GPT (or another LLM) to do this labeling task cheaply and at scale. The challenge is that unless these labels are _always correct_, then the coefficients in your downstream analysis will have unknown bias and have incorrect confidence intervals.

We offer a framework to leverage these LLM labels while also ensuring asymptotic consistency and valid confidence intervals, which we dub **design-based supervised learning**. A brief summary:

- The researcher samples a subset of the documents for expert labeling (something we argue that many researchers would do anyways to assess LLM performance).
- The researcher uses the known probability of sampling to "upweight" error corrections in the labeled subset to construct a bias-corrected psuedo-outcome.
- Using bias-corrected psuedo-outcome in downstream analyses ensures consistency and asymptotic normality.

Proofs and experiments are shown in the paper, and software and short video explainers are coming soon!


### Resources:

- [Link to Preprint](https://doi.org/10.48550/arXiv.2306.04746)
- [Link to Supplement](https://naokiegami.com/paper/dsl_supplement.pdf)
- [Link to 5min Video Explainer](https://recorder-v3.slideslive.com/?share=88028&s=fdfbd1be-a7fd-40b3-9cfe-2636d0ffe0e5)
