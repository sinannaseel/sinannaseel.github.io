---
layout: default
title: Study Notes
---

# Study Notes

My learning journey and revision notes on topics relevant to my research.

## Core Topics

{% assign core_topics = site.notes | where: "category", "core" %}
{% for note in core_topics %}
- [{{ note.title }}]({{ note.url | relative_url }}){% if note.excerpt %} - {{ note.excerpt | strip_html | truncatewords: 15 }}{% endif %}
{% endfor %}

## Rearrangement Techniques

{% assign rearrangement = site.notes | where: "category", "rearrangement" %}
{% for note in rearrangement %}
- [{{ note.title }}]({{ note.url | relative_url }})
{% endfor %}

## Specialized Topics

{% assign specialized = site.notes | where: "category", "specialized" %}
{% for note in specialized %}
- [{{ note.title }}]({{ note.url | relative_url }})
{% endfor %}