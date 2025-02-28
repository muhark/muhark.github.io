---
title: 'LLaVA-Gemma: Accelerating Multimodal Foundation Models with a Compact Language Model'
authors: 'Musashi Hinck*, Matthew L Olson*, David Cobbley, Shao-Yen Tseng, Vasudev Lal'
collection: publications
category: workshops
permalink: /papers/cvpr-mmfm-2024-llavagemma
excerpt: 'We train a suite of multimodal foundation models (MMFM) using the popular LLaVA framework with the recently released Gemma family of large language models (LLMs).'
date: 2024-02-26
venue: 'CVPR 2024 Multimodal Foundation Models Workshop'
# slidesurl: ''
paperurl: 'https://arxiv.org/abs/2404.01331'
highlight: "false"
order: 99
---

- Featured on HuggingFace Daily Papers! [Tweet](https://x.com/_akhaliq/status/1775348024278962373)
- Model on HuggingFace: [https://huggingface.co/Intel/llava-gemma-2b](https://huggingface.co/Intel/llava-gemma-2b).

## Abstract

We train a suite of multimodal foundation models (MMFM) using the popular LLaVA framework with the recently released Gemma family of large language models (LLMs). Of particular interest is the 2B parameter Gemma model, which provides opportunities to construct capable small-scale MMFMs. In line with findings from other papers in this space, we test the effect of ablating three design features: pretraining the connector, utilizing a more powerful image backbone, and increasing the size of the language backbone. The resulting models, which we call LLaVA-Gemma, exhibit moderate performance on an array of evaluations, but fail to improve past the current comparably sized SOTA models. Closer analysis of performance shows mixed effects; skipping pretraining tends to reduce performance, larger vision models sometimes improve performance, and increasing language model size has inconsistent effects. We publicly release training recipes, code and weights for our models for the LLaVA-Gemma models.