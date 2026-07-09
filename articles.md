---
layout: default
title: "Articles"
permalink: /articles/
---

{% assign visible_articles = site.articles | where_exp: "a", "a.draft != true" %}
{% assign sorted_articles = visible_articles | sort: 'date' | reverse %}
{% assign categories = sorted_articles | map: 'category' | uniq %}
{% assign excluded = "Life,Book Reviews" | split: "," %}

{% for category in categories %}
{% if excluded contains category %}{% continue %}{% endif %}
## {{ category }}

{% for article in sorted_articles %}
{% if article.category == category %}
- [{{ article.title }}]({{ article.url }})
{% endif %}
{% endfor %}
{% endfor %}
