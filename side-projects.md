---
layout: default
title: "Side Projects"
permalink: /side-projects/
---

Things I built outside of work — usually with someone else, usually not aimed at being a business. Each post walks through the concept, what got built, and an interactive demo where it makes sense.

{% assign visible_projects = site.side_projects | where_exp: "p", "p.draft != true" %}
{% assign sorted_projects = visible_projects | sort: 'date' | reverse %}

{% for project in sorted_projects %}
- [{{ project.title }}]({{ project.url }}) <span class="badge">Published {{ project.date | date: "%B %-d, %Y" }}</span>
{% endfor %}
