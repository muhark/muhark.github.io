---
layout: project_single
title: "Estimating the Effect of Micro-Targeting"
slug: "does-microtargeting-work"
---

Back in October 2020, I conducted a survey experiment in the US designed to answer the following question:

	_Does micro-targeting work?_

Participants in this survey were asked a number of questions about demographics and political opinions, then shown one of five possible anti-Biden advertisement run by the Trump campaign. Finally their opinion of Biden and Trump, and their voting intentions were recorded.

Participants in the control group were assigned an advertisement at random, and this data was used to train an algorithm that targeted participants in the treatment group with the advertisement predicted to be the most effective based on their answers to the pre-treatment questions.

The results were alarming. Among participants who identified with neither party, and had not pre-voted at the time of the survey, targeting:

- Increased the proportion of participants reporting that they dislike Biden by 8.7 percentage points.
- Decreased the proportion of participants reporting that they intend to vote for Biden by 7.1 percentage points.

All differences can be seen in the figure above. These values are relative to those who received an advertisement at random.

### Resources:

- [Link to Paper](/static/data/docs/harukawa-2021-microtargeting.pdf)
- [Link to GitHub Repo](https://github.com/muhark/dotas-design)

