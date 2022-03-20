---
layout: post
title: "On Emotionality and Reason in Political Language (Gennaro and Ash 2021)"
date: 2022-03-19 00:00:00 +0000
categories: [reading, ml, nlp]
---

It's been a little while since I've had time to update this blog. A bit of personal news/what I've been up to in that time:

- I attended the TADA2021 conference at University of Michigan where I presented a poster on [Multimodal Transformers](https://muhark.github.io/project/multimodal-ad-transformers/).
- I started a pre/post-doc at University College London with Professor Lucy Barnes on the UKRI-funded MENMOPE project.
- I completed work on another paper applying interpretable deep learning techniques to characterizing language usage by political campaigns. (_More on this soon!_)

I probably overestimated my ability to keep this blog regularly updated with long posts as the end of my doctorate rapidly approaches, so this will be a short post. I want to share some thoughts I had about a brilliant paper I recently had the pleasure to read.

## Emotion and Reason in Political Language

In a [recently published article](https://elliottash.com/wp-content/uploads/2021/08/Emotions_in_Politics_Economic_Journal_CA.pdf), Gennaro and Ash propose a (very cool) method for measuring _pathos_ and _logos_ in political speech. To briefly summarise (what I understood of) their paper and method:

The objective is to measure the extent to which a political speech employs emotional versus rational language. This dichotomy has roots in Aristotelian thought, and I think it is widely employed in academic, professional, and casual (i.e. "common-sense") settings.

The authors begin with two lexicons (i.e. word lists) from LIWC (Pennebaker et al 2015) to capture "rational" and "emotional" language. They then train a Word2Vec model (Mikolov et al 2013) on the US Congressional Speech Corpus (roughly 10M speeches). Some pre-processing choices include retaining only nouns, adjectives and verbs, and using the Snowball stemmer.

To produce document embeddings from the resulting word vector space, they take the average of the individual word vectors of a document weighted by the inverse frequency of the word plus a constant smoothing parameter. "Poles" for emotionality and rationality are constructed similarly using the aforementioned LIWC lexicons.

The resulting measure of emotionality or rationality for a given speech is the cosine distance between the document vector of the speech and the corresponding "pole". The authors provide extensive validation of their measure including (importantly!) correlation with human judgment.

### The Intuition (I think)

People vary in what they find "intuitive", especially when it comes to quantitative methods. I found myself frowning at what I found to be a slightly "hand-wavey" explanation of why the method works the way it does:

>  A word, normally a string object drawn from a high-dimensional list of categories, is “embedded” in a lower-dimensional space, where the geometric location encodes semantic meaning.

We know broadly that this holds, but the phrase "geometric location encodes semantic meaning" is doing a lot of the heavy lifting for the link between the theory and empirics. I get that this may be a necessary simplification to stay out of the methodological bushes, but this kind of oversimplification can lead to a problematic lack of nuance/precision in downstream analysis.

This is to say, even though I am sure that the authors Gennaro and Ash have a good sense of what they can and cannot claim on the basis of their measure, what they have measured is only a particular sense of emotionality and rationality in political language, and cannot be applied as broadly or flexibly as a theoretical construct like _pathos_ or _logos_.

Here's how I would explain the intuition behind their method, why it works, and what I think they have measured. Recall [how we generate Word2Vec embeddings in my last post](https://muhark.github.io/python/ml/nlp/2021/10/21/word2vec-from-scratch.html). Word2Vec embeddings are _essentially_ (here's my comfortable level of hand-waving) a dense representation of the conditional probabilities of a word given its context (or vice versa). When we say that it encodes semantic meaning, this is a distributional notion of lexical semantics that words with similar meaning occur in similar contexts.

Remember that the authors remove all words other than nouns, verbs and adjectives. Thus their word vectors are a dense representation of co-occurrence of particular nouns/verbs/adjectives in the same speeches. It generally holds that vectors of words with similar contexts are encoded to have similar (vector) direction. (I've been wondering why this follows from the autoencoder architecture, but haven't grokked it yet. It makes sense to me that encoding similar words in similar spaces would result in a more "efficient" classifier, but I don't understand why the loss function will converge in this direction.)

In this space, we represent speeches and "poles" as a weighted average of the individual word vectors. Given that nouns/verbs/adjectives occurring in similar contexts are encoded to similar directions, we can think of this approach as a "fuzzy continuous dictionary" logic. Fuzzy because we include words that aren't in the lexicons, but occur in similar contexts, and continuous because our final measure of cosine distance is continuous.

Will this measure emotionality/rationality? To the extent that emotionality and rationality in political speech can be described as phenomena occuring at the lexical and semantic level, Gennaro and Ash provide a sensible strategy for flexibly measuring the similarity of documents to a theoretically motivated tokens.

On the other hand, I'm not sure that I think of emotionality and rationality being solely or even primarily lexical/semantic phenomena. The perception of rationality in the listener, for instance, is constructed partially by lexical choices (and the meaning of those words), but more imporantly it exists in the structuring of the sentence (syntax, or in the language of LFG the **c-structure** and **d-structure**). When I think of speeches conveying "rationality", I think primarily of the use of deduction, entailment, and so on. Emotionality may be more describable by lexical semantics, but likewise I think that much of emotionality in practice exists in prosody (**p-structure**) and listener-side effects (such as perceived gender or race). 

Is this a problem for Gennaro and Ash's method? I'm not sure. A cautious take would be: "given that their measure does not directly measure the linguistic phenomena that produce a perception of emotionality/rationality, but rather strongly correlates with it because of the association between lexical semantics and syntax/prosody etc., their proof is purely inductive and therefore we have not established the conditions under which their method will fail". On the other hand I reject the notion that the onus is on the researcher to define the precise scope conditions of their method in all cases, especially where those bounds are difficult (or impossible) to deductively establish. However, in the absence of that elusive perfect scenario where what we want to measure in theory and what we are able to measure in practice are the exact same, clear explication of the derivation of your measure is crucial to avoid "abuse" of the measure in downstream analyses.

I hope that the authors do not take any of this as an attack on their work–I thoroughly enjoyed reading and thinking about this paper, and hope political science continues to move in this direction.

### A small aside on vector norms:

_Why not take the vector norm before averaging word vectors_? I discussed this question with a colleague specifically in terms of whether this would bias the results; in short I don't think it does, but I still think it's an odd choice. Here's why.

Magnitude can be thought of like a weight when averaging vectors, i.e. longer vectors are more heavily weighted. It's clear that GA21 have in mind that not all words should be equally weighted when constructing document vectors, which is why they weight by inverse word frequency.

This makes sense! You want to reduce the signal due to common/ubiquitous words, and amplify the signal of distinguishing words. For the exact same reason, it doesn't make sense to me that they don't norm the same vectors before constructing document vectors, because I don't think we have a strong sense what those "weights" substantively _mean_.

A few papers discuss this. [Schakel and Wilson (2015)](https://arxiv.org/pdf/1508.02297.pdf) argue that word vector magnitude corresponds to some sense of "significance". If this "significance" corresponds to some notion of salience/distinguishing-ness that is theoretically useful, then not norming the vectors is benign (and probably helpful). Likewise, [Kozlowski et al (2019)](https://journals.sagepub.com/doi/full/10.1177/0003122419877135) make reference in a footnote to the decreasing ratio between the surface area and volume of hyperspheres. I think that they are saying that this means we can ignore the difference between normed and non-normed vectors? Ultimately this can be answered with a simple robustness check, and it's possible I missed that in the supplementary materials.

