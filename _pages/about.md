---
permalink: /
title: ""
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

I am a Research Scientist in the Emergent Artificial Intelligence Lab at Intel Labs.

The substantive focus of my research is characterizing latent social biases in large generative models and techniques for correcting/steering these biases.

In practice, my work is up and down the AI Research/Engineering stack, from optimizing training for large multimodal models on hundreds of accelerators, to writing custom models and pipelines, to designing experiments and evaluation metrics.

Prior to Intel, I was postdoc at Princeton University working with [Professor Brandon Stewart](https://bstewart.scholar.princeton.edu), and did my PhD at the University of Oxford with Professors [Andy Eggers](https://andy.egge.rs) and [Raymond Duch](https://www.raymondduch.com).


## News

- **Feb 2025**: New dataset on HuggingFace: [IssueBench](https://huggingface.co/datasets/Paul/IssueBench)
- **Dec 2024**: Scholar Award for top Intel Labs Academic Author
- **Dec 2024**: Spotlight Paper at the Creativity and AI Workshop at NeurIPS 2024
- **Oct 2024**: 3 papers accepted to NeurIPS 2024 Workshops
- **Sep 2024**: 2 papers accepted to EMNLP 2024
- **Aug 2024**: [ACL Outstanding Paper Award](https://arxiv.org/pdf/2402.16786)!
- **Mar 2024**: New model on HuggingFace: [intel/llava-gemma-2b](https://huggingface.co/Intel/llava-gemma-2b)
- **Feb 2024**: Started at Intel Labs as AI Research Scientist

## Publications
_For a full list of papers, see [here](/papers)_.

### Highlights

<ol>{% assign sorted_papers = site.papers | sort: 'order' %}
  {% for post in sorted_papers %}
    {% if post.highlight == "true" %}
      {% include archive-single-publication.html %}
    {% endif %}
  {% endfor %}</ol>

### Preprints 

<ul>{% assign sorted_papers = site.papers | sort: 'order' %}
  {% for post in sorted_papers %}
    {% if post.category == "preprints" %}
      {% include archive-single-publication.html %}
    {% endif %}
  {% endfor %}</ul>


