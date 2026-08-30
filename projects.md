---
layout: default
title: "Projects"
permalink: /projects/
nav: projects
---

{% assign projects = site.writing | where_exp: "p", "p.draft != true" %}
{% assign projects = projects | where_exp: "p", "p.tags contains 'Side Project'" %}
{% assign projects = projects | sort: 'date' | reverse %}
<div class="project-grid">
{% for post in projects %}
<a class="project-card" href="{{ post.url }}">
  <div class="project-thumb"{% unless post.thumb_image %} style="background:{{ post.thumb_color | default: 'var(--color-accent)' }};"{% endunless %}>
    {% if post.thumb_image %}
    <img src="{{ post.thumb_image }}" alt="">
    {% else %}
    <span class="project-thumb-letter">{{ post.title | slice: 0 }}</span>
    {% endif %}
  </div>
  <div class="project-body">
    <div class="row-date">{{ post.date | date: "%b %Y" }}</div>
    <div class="row-title">{{ post.title }}</div>
    <div class="row-tags">{% for tag in post.tags %}{% unless tag == "Side Project" %}<span class="row-tag">{{ tag }}</span>{% endunless %}{% endfor %}</div>
  </div>
</a>
{% endfor %}
</div>
