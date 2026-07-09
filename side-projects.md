---
layout: default
title: "Side Projects"
permalink: /side-projects/
nav: side-projects
---

{% include page-heading.html title=page.title %}
<p>Things I built outside of work, sometimes with someone else. Each post walks through the concept, what got built, and an interactive demo where it makes sense.</p>
{% assign visible_projects = site.side_projects | where_exp: "p", "p.draft != true" | sort: 'date' | reverse %}
<div class="listing">
{% for project in visible_projects %}
<a class="listing-row" href="{{ project.url }}">
<div class="row-date">{{ project.date | date: "%b %Y" }}</div>
<div class="row-title">{{ project.title }}</div>
<div class="row-excerpt">{{ project.excerpt | strip_html | truncatewords: 24 }}</div>
</a>
{% endfor %}
</div>
