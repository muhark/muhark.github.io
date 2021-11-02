---
layout: project_single
title: "Multimodal Transformer Strategies for Political Ads"
slug: "multimodal-ad-transformers"
---

## Improving Text Embeddings with Images?

- Initial social science applications of word embeddings focus on their use for describing changing semantic relations (Garg et al 2018, Rodman 2019).
- Recent works use embeddings to operationalize text data for downstream tasks: scaling (An et al 2018), regression (Rodriguez, Spirling and Stewart 2021), and community detection (Rheault and Musulan 2021).
- Given that we are not interested in individual vectors, we do not need to restrict ourselves to media with interpretable units.
- I investigate how and whether we can operationalize multimodal data sources for downstream supervised learning tasks.

## Dataset and Task

- Leverage existing datasets of political ads with human-labelled or latent features that affect the content without being directly observable. Poster shows results using subset of ProPublica Political Facebook Ads dataset.
- Restrict to sponsors with at least 100 observations (N=44,162), split 9-1-90% train-validation-test.
- Compare models trained+validated on 10% of the data on their ability to predict the sponsor of the remaining 90% of ads from the text and/or image data.

## Why Transformers?

- Transformer-based (Vaswani et al. 2017) encoder models such as BERT (Devlin et al. 2018) are dominant in NLP tasks.
- Encoder models can be pre-trained on large corpus then fine-tuned on task-specific data.

![](https://raw.githubusercontent.com/muhark/muhark.github.io/master/static/projects/multimodal-ad-transformers/comparison.png)

## Performance and Outcomes

![](https://raw.githubusercontent.com/muhark/muhark.github.io/master/static/projects/multimodal-ad-transformers/joint-v-sep.png)

### Multimodality

- Image+Text approach yields strongest accuracy (78%, N labels = 150).
- But only modest 2.7% improvement vs Text-only.

### By Sponsor Type:

- Strongest overall performance on Misc category, which included Penzeys Spices, Counterpoint cartoons and Dissent Pins.
- Largest gain vs. Text-only for  Conservative PAC/Orgs (16.2% relative increase).
- No uniform pattern for weak Image-only resulting in weak Joint.

## Next Steps

- Predicting tone, topic and style in political  ad videos using ensemble of unimodal transformers.
- Probing for qualitative understanding of difference in performance.

#### References

1. Garg, Nikhil, Londa Schiebinger, Dan Jurafsky, James Zou. "Word embeddings quantify 100 years of gender and ethnic stereotypes".  PNAS (2018): 115-16.
2. Rodman, Emma. "A Timely Intervention: Tracking the Changing Meanings of Political Concepts with Word Vectors". Political Analysis (2020): 28-1.
3. An, Jisun, Haewoon Kwak, and Yong-Yeol Ahn. "Semaxis: A lightweight framework to characterize domain-specific word semantics beyond sentiment." arXiv preprint arXiv:1806.05521 (2018).
4. Rodriguez, Pedro L., Arthur Spirling, and Brandon M. Stewart. "Embedding Regression: Models for Context-Specific Description and Inference." Working Paper (2021).
5. Rheault, Ludovic, and Andreea Musulan. "Efficient detection of online communities and social bot activity during electoral campaigns." Journal of Information Technology & Politics (2021): 1-14.
6. Vaswani, Ashish, et al. "Attention is all you need." Advances in NeurIPS. (2017).
7. Devlin, Jacob, et al. "Bert: Pre-training of deep bidirectional transformers for language understanding." arXiv preprint arXiv:1810.04805 (2018).
