---
layout: post
title: "DAGs vs PO: Notes on Imbens (2020)"
date: 2021-04-08 11:00:00 +0100
categories: [causinf, reading]
---

A couple of years ago, while taking Andy Eggers/Spyros Kosmidis' Causal Inference course, I had a conversation with a theoretical physicist about causality. They were shocked that I had not heard of Judea Pearl, but in turn they had never heard of Instrumental Variables, Diff-in-Diff or RDD.

In the follow up to this conversation, I read up on Directed Acyclic Graphs (DAGs) and realised that I had been taught them informally in my undergraduate Political Sociology tutorials (with Sergi Pardos-Prado), but had never learned about the maths behind them.

From this, and later conversations about Pearl and Directed Acyclic Graphs (DAGs), I was given the following takeaways that I couldn't make much sense of:

1. DAGs are diametrically opposed to the Potential Outcomes framework.
2. DAGs are the final solution to causal inference.

I assumed the latter claim was a bit of meaningless hype, but I couldn't figure out what in all that I had learned about the Potential Outcomes framework was contradictory to Pearl.

Fast forward to a week ago, when I came across a paper by [Guido Imbens](https://imbens.people.stanford.edu/) titled "Potential Outcome and Directed Acyclic Graph Approaches to Causality: Relevance for Empirical Practice in Economics". This article helped clarify my confusion around the former claim a great deal. (Spoiler: they're not diametrically opposed, but they are different).

Here is a summary of some of the my main takeaways and thoughts on the paper. It's a great paper to read if:

- You want to know what the PO framework has to offer over the DAG framework.
- You want a quick summary/refresher on the DAG and PO framework.

My institution gave me access to the final published version, but in case yours doesn't, [here is the link to the pre-print](https://www.nber.org/system/files/working_papers/w26104/w26104.pdf).

## **tl;dr**

This paper summarises and compares the two main frameworks for _causal inference in observational studies_:

- **Directed Acyclic Graphs (DAGs)**: associated with Pearl et al.
- **Potential Outcomes (PO)**: associated with Rubin et al.

and provides some suggestions as to why the PO framework is more mainstream in econometrics (and related social sciences). To this effect, Imbens works through six specific scenarios where the PO framework offers a way forward that DAGs cannot.

### An aside for my readers wondering why and what _causal inference_ is

We often observe (or induce) pairs of events that we want to connect with a mechanism; $X$ _causes_ $Y$. Despite our use of it as an everyday concept, _causality_ is complicated and profound, and is embedded in our societies and understanding of reality (how would our legal system work without causality?)

A great many research questions are interested in the _causal effect_ of some phenomenon/treatment. Examples range from assessing the impact of a medicine on an illness to the impact of a policy on a social condition, such as poverty.

You've probably heard, and maybe even said in retort, the common refrain "correlation doesn't imply causation". That's certainly true, but our attempts to understand these events shouldn't end there. There's a sizable literature on showing when and how we can make claims about causal relationships between events. This article is discussing two of the main streams of literature addressing this task.

### A Very Brief Intro to DAGs and PO

I won't provide an in-depth explanation of DAGs and PO here, since the article already does a great job of summarising key bits of an extensive literature. To heavily oversimplify for those short on time, the DAG approach essentially consists of the following.

Given a query about a causal process, you first:

1. Identify the variables in your causal model (and draw them as nodes, i.e. circles).
2. Identify the theoretical relationships between your variables (and draw them as directed edges, i.e. arrows).

There are some restrictions on the resulting graph, e.g. no cycles X→Y→Z→X...

Once you have this graph, it allows you to systematically identify _unconfounded_ relationships (through a tool called _do_-calculus).

In contrast, the PO framework is characterized by a focus of the same outcome under different states. Suppose that $Y$ is your outcome, $D$ is a (usually, but not necessarily) binary treatment. We are interested in comparing the values of the outcome for the same unit under both treatment conditions: $Y(0)$ and $Y(1)$. Unfortunately, for that unit, you only ever observe one of potential outcomes (something referred to as the "fundamental problem of causal inference" (Holland 1986)).

However, it is possible to obtain an unbiased estimate of the average difference between $Y(0)$ and $Y(1)$ for a given group. The simplest way to do this is to randomly assign units to the treatment ($D=1$) and control ($D=0$) groups, so that there is no systematic difference between the groups other than the treatment.

As noted by Imbens (and a great many others), it is often not possible to conduct a randomized experiment. However, using the PO framework, it is possible to identify situations and designs that satisfy the requirements of an unbiased estimator of the average treatment effect by leveraging real-world randomness. Some key designs/examples of these techniques include:

- Instrumental Variables
- Differences in Differences
- Regression Discontinuity Design

For a friendly and quick introduction to these and PO, I recommend Mastering 'Metrics or Mostly Harmless Econometrics by Angrist and Pischke.


## Imbens on DAGs vs PO

In Imbens' words, the primary focus of the DAG approach to causality is _identification_, rather than estimation and inference. In short, a researcher comes armed with a number of variables, some observed and some unobserved, and a theoretical model linking the causal relationships between these variables.

In terms of how DAGs can be used to answer causal questions, Imbens offers the following:

>	if you tell me how the world the world works (by giving me the full causal graph), I can tell you the answers.

As he later notes, the utility of this approach depends in part on the extent to which the researcher is willing to assume that their model truthfully captures reality. This might seem a bit trivial in that it's a requirement most deductive empirical research faces, but the PO framework does allow us to build up to qualified claims with _prima facie_ implausibility.

In short, Imbens argues that DAGs provide two main benefits:

- DAGs help you explicitly formulate the assumptions underpinning your causal model.
- The _do-calculus_ developed in the DAG literature allows for systematic assessment of particular kinds of causal queries.

I think the first point is the most often-cited one, and is quite important. For many, depicting your model visually makes it easier to clarify your assertions, and also recognize pathways that you may not be accounting for.

In contrast, some of the benefits of the PO framework include:

- PO framework better captures particular key assumptions, namely monotonicity and other shape restrictions.
- Identification strategies in PO focus on models with relatively few variables.
- PO lends itself well to accounting for treatment effect heterogeneity.
- PO literature linked to a lot of practical advice on design and implementation.

In particular, Imbens highlights the following as the reason for the relative lack of adoption of the DAG approach in empirical economics (and social sciences more broadly).

>	[...] another reason for the lack of adoption in economics is that the DAG literature has not shown much evidence of the alleged benefits for empirical practice in settings that resonate with economists.

>	the DAG literature [...] have no substantive empirical examples, focusing largely on identification questions in what TBOW [The Book of Why] refers to as "toy" models.

> 	the toy models in the DAG literature sometimes appear to be a set of solutions in search of problems, rather than a set of clever solutions for substantive problems previously posed in social sciences.

I'm not sure whether the empirical approaches above (IV, DiD, RDD) are necessarily _of_ the PO framework. However, the PO framework does help express these approaches and their assumptions clearly, which is helpful to empirical researchers. Imbens does note that it is possible to diagram IV with DAGs, but this misses some of the key assumptions (namely monotonicity).

## Some Theoretically Interesting Tidbits

I'm adding a few additional bits, focused on metaphysical or epistemological questions, that I found interesting in this article.

### Manipulability of Causes

In section 4 of this article, Imbens discusses six specific issues with the DAG framework hampering its suitability to questions asked by economists. I found this one particularly interesting:

The literature surrounding the PO framework emphasises the _manipulability of causes_. In simple terms, this means causal effects should not be tied to non-manipulable attributes. Taking the example of the effect of a job applicant's race or gender; instead of imagining what it would mean to _ceteris paribus_ change the race or gender of an individual, an experiment should tie the causal effect to something manipulable, such as the effect of the hiring manager's knowledge of the applicant's race or gender. 

Imbens argues that this distinction matters, because how an individual ends up at the treated or untreated state matters. Discussing the example of the effect of obesity on health, Imbens points out that how an individual becomes obese/non-obese (e.g. through diet, exercise, or surgery) matters for the outcome. In contrast, Pearl argues that it is perfectly plausible to imagine _do(obesity=x)_, "_by which nature sets obesity to $x$, independent of diet or exercise, while keeping every thing else intact, especially the processes that respond to $X$_".

I think I understand what Pearl is getting at, especially if we are thinking of average effects (and not individual ones). Equally, I would think that one should at least concede that it would be more useful to separate the heterogeneous effects of non-obesity, conditional on the method of becoming not obese. In that way it seems like a more pragmatic "requirement" of the PO framework, and one that is due to the PO framework's focus on policy, rather than an inherent requirement of the framework. (None of which contradicts anything Imbens says in this article).

### Describing Assumptions/Restrictions of IV and RDD

This point is made throughout the article, but in brief: it is difficult to formalise the assumptions required for IV and RDD with DAGs. Namely, for IV this is the monotonicity restriction, and for RDD this is discontinuity in one conditional expectation + smoothness of other conditional expectations.

Here the point to me seems to be more refuting the supremacy of DAGs; they are not adequate for explicating certain important empirical tools for causal inference.

### A "Fourth Rung" on the Ladder of Causality

Pearl introduces a classification of causal problems that he calls the _ladder of causality_, with three rungs:

1. Association: association between aspirin and headaches
2. Intervention: what would happen to my headache if I take an aspirin?
3. Counterfactuals: given that I took the aspirin, what would've happened had I not taken the aspirin?

Imbens notes that PO framework is generally apprehensive of answering third-rung questions that depend on individual-level heterogeneity. In contrast, these kinds of questions are frequently discussed by lawyers ("but for" analyses).

Finally, Imbens notes a further fourth rung: "'why', or reverse causality questions". In my undergraduate I mostly studied political science works that addressed "why" questions, such as "why did Chile transition to democracy?" Since starting research, and focusing on quantitative/causal inference methods, I realise that I answer such questions in an indirect way: "what is the average effect of _cause X_ on _outcome Y_?".

