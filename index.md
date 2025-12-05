---
layout: default
title: Home
---

# Welcome

I am working on robotic manipulation in confined spaces using Task and Motion Planning (TAMP) for rearrangement of items. My research focuses on developing algorithms that enable robots to efficiently reorganize objects in cluttered, constrained environments.

## Current Research

- Task and Motion Planning for object rearrangement
- Optimization in confined spaces with physical constraints
- Entropy-based measures for clutter quantification
- Non-monotonous and sequential rearrangement strategies

[Read more about my research →](/research)

## Recent Notes

{% for note in site.notes limit:5 %}
- [{{ note.title }}]({{ note.url | relative_url }}) {% if note.date %}- {{ note.date | date: "%B %d, %Y" }}{% endif %}
{% endfor %}

[View all notes →](/notes)