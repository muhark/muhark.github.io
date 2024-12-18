---
title: "Debiasing Large Vision-Language Models by Ablating Protected Attribute Representations"
authors: "Neale Ratzlaff, Matthew Lyle Olson, Musashi Hinck, Shao-Yen Tseng, Vasudev Lal, and Phillip Howard"
collection: publications
category: conferences
permalink: /publication/mechinterp-workshop-neurips-2024
excerpt: ''
date: 2024-10-17
venue: 'NeurIPS 2024 SafeGenAI Workshop'
paperurl: 'https://arxiv.org/abs/2410.13976'
---


## Abstract

Large Vision Language Models (LVLMs) such as LLaVA have demonstrated impressive capabilities as general-purpose chatbots that can engage in conversations about a provided input image. However, their responses are influenced by societal biases present in their training datasets, leading to undesirable differences in how the model responds when presented with images depicting people of different demographics. In this work, we propose a novel debiasing framework for LVLMs by directly ablating biased attributes during text generation to avoid generating text related to protected attributes, or even representing them internally. Our method requires no training and a relatively small amount of representative biased outputs (~1000 samples). Our experiments show that not only can we can minimize the propensity of LVLMs to generate text related to protected attributes, but we can even use synthetic data to inform the ablation while retaining captioning performance on real data such as COCO. Furthermore, we find the resulting generations from a debiased LVLM exhibit similar accuracy as a baseline biased model, showing that debiasing effects can be achieved without sacrificing model performance.