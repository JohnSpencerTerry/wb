---
layout: default
title: "Articles"
permalink: /articles/
---

{% assign sorted_articles = site.articles | sort: 'date' | reverse %}
{% assign categories = sorted_articles | map: 'category' | uniq %}

{% for category in categories %}
## {{ category }}

{% for article in sorted_articles %}
{% if article.category == category %}
- [{{ article.title }}]({{ article.url }})
{% endif %}
{% endfor %}
{% endfor %}
