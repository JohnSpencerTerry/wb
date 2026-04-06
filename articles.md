---
layout: default
title: "Articles"
permalink: /articles/
---

{% assign sorted_articles = site.articles | sort: 'path' %}
{% for article in sorted_articles %}
- [{{ article.title }}]({{ article.url }})
{% endfor %}
