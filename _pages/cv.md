---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

# Education

- DPhil University of Oxford, 2022
- MSc University of Oxford, 2019
- BA University of Oxford, 2015

# Employment

- **AI Research Scientist**, Intel Labs, USA (2024-present)
- **Postdoctoral Research Associate**, Princeton University, NJ, USA (2022-2024)
- **Quantitative Analyst**, SBI Group, Tokyo, Japan (2016-2018)

# Publications

<ul>
{% for post in site.publications reversed %}
    {% include archive-single.html %}
{% endfor %}
</ul>
  
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