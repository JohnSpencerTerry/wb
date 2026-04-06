---
layout: default
title: Articles
---


{% assign sorted_articles = site.articles | sort: 'path' %}
<ul>
  {% for article in sorted_articles %}
    <li><a href="{{ article.url }}">{{ article.title }}</a></li>
  {% endfor %}
</ul>