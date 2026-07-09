---
layout: default
title: "Articles"
permalink: /articles/
nav: articles
---

{% include page-heading.html title=page.title %}
{% assign visible_articles = site.articles | where_exp: "a", "a.draft != true" %}
{% assign sorted_articles = visible_articles | sort: 'date' | reverse %}
{% assign categories = sorted_articles | map: 'category' | uniq %}
{% assign excluded = "Life,Book Reviews" | split: "," %}
<div class="listing">
{% for category in categories %}
{% unless excluded contains category %}
{% assign in_cat = sorted_articles | where: "category", category %}
<div class="listing-group">
{% for article in in_cat %}
<a class="listing-row" href="{{ article.url }}">
<div class="row-date">{{ article.date | date: "%b %Y" }}</div>
<div class="row-title">{{ article.title }}</div>
<div class="row-excerpt">{{ article.excerpt | strip_html | truncatewords: 24 }}</div>
</a>
{% endfor %}
</div>
{% endunless %}
{% endfor %}
</div>
