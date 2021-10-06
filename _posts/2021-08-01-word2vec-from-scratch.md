---
layout: post
title: "Implementing Word2Vec in PyTorch"
date: 2021-08-01 00:00:00 +0000
categories: [python, ml, nlp]
---

Eight years since [Mikolov et al](https://arxiv.org/abs/1301.3781) first proposed a novel architecture for computing vector representation of words in 2013, I think we can safely say that the "word embedding" approach to operationalising text data is entering the political science text-as-data methods mainstream.

The utility of embedding methods is linked directly to the original challenges motivating text-as-data methods. By representing natural language numerically, embedding methods offer the possibility to leverage a broad range of quantitative tools on hitherto unusable sources of data.

The number of political science papers using word embeddings has exploded in just the past three years. It's not my intention to write a full-on literature review in this post, but these three articles (in my humble opinion) are an excellent place to start in understanding the potential applications of word embedding models for computational social science.

- [Garg et al (2018), PNAS](https://www.pnas.org/content/pnas/115/16/E3635.full.pdf) use word embeddings to quantify shifting gender and ethnic stereotypes in the US over the past 100 years.
- [Rodman (2019), Political Analysis](https://static1.squarespace.com/static/5ca7d04ea09a7e68ba44e707/t/5cda219af4e1fc94236bc0cf/1557799325771/Diachronic_Word_Vectors___Political_Analysis_Final_Version.pdf) uses word embeddings to look at the changing meaning and context of salient terms such as "equality" in US news corpora.
- [Rodriguez and Spirling (2021), Journal of Politics](https://www.journals.uchicago.edu/doi/10.1086/715162) evaluate the utility of word embedings for various social science applications.

At a high level, word embeddings represent the individual words (vocabulary) of a collection of texts (corpus) as vectors in a _k_-dimensional space (where _k_ is determined by the researcher–more on this later). These vectors encode information about the relationship between words and their context, and are used for downstream language modelling tasks.

If you're crippled by the same induction-skeptical methods anxiety that I am, then you too may be wondering:

- How does it work?
- Why does it work?
- How do we know whether it's worked?

In order to answer the first two questions for myself, I recently tried implementing my own version of Mikolov et al's `Word2Vec` algorithm in PyTorch. (Note that the state-of-the-art has moved past `Word2Vec` in Natural Language Processing, and I suspect that computational social science will follow suit soon. Nevertheless, implementing papers in code is always a good exercise.)

I'll try to write in a way that is accessible to researchers without familiarity with Python, PyTorch or neural networks. As usual, feel free to reach out to me on Twitter, etc. if you want more clarity on anything in this post.

# Step 1: DataLoader

As with any NLP task (or any data analysis task for that matter), there are two steps:

- Preparing Data (the Loader)
- Processing Data (the Model)

```{python}
# %%
import torch
import datasets

dataset = datasets.load_dataset('tweets_hate_speech_detection')

text = dataset['train']['tweet'][0]

# %%
# For simplicity let's remove alphanumeric but keep @, #
import re

def split_tokens(text):
    return re.split(r" +", re.sub(r"[^A-z#@ ]", "", text.lower()))

# %%
# Determine vocabulary so we can create mapping
dataset = dataset.map(
    lambda row: {'tokens': split_tokens(row['tweet'])})

# %%
from collections import Counter

counts = Counter([i for s in dataset['train']['tokens'] for i in s])
counts.most_common()



```
