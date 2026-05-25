---
layout: default
title: "Articles"
permalink: /articles/
---

{% assign visible_articles = site.articles | where_exp: "a", "a.draft != true" %}
{% assign sorted_articles = visible_articles | sort: 'date' | reverse %}
{% assign categories = sorted_articles | map: 'category' | uniq %}

{% for category in categories %}
## {{ category }}

{% for article in sorted_articles %}
{% if article.category == category %}
- [{{ article.title }}]({{ article.url }}) <span class="badge">Published {{ article.date | date: "%B %-d, %Y" }}</span>
{% endif %}
{% endfor %}
{% endfor %}
