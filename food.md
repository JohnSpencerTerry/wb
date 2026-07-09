---
layout: default
title: "Food"
permalink: /food/
nav: food
---

{% include page-heading.html title=page.title %}
<p>Recipes  plus notes from cooking, shopping, and eating.</p>
{% assign visible_posts = site.food | where_exp: "p", "p.draft != true" %}
{% assign sorted_posts = visible_posts | sort: 'date' | reverse %}
{% assign ordered = "Recipes,Kitchen Notes,NYC Eats" | split: "," %}
{% assign all_categories = sorted_posts | map: 'category' | uniq %}
<div class="listing">
{% for category in ordered %}
{% assign in_cat = sorted_posts | where: "category", category %}
{% if in_cat.size > 0 %}
<div class="listing-group">
<div class="listing-group-label">{{ category }}</div>
{% for post in in_cat %}
<a class="listing-row" href="{{ post.url }}">
<div class="row-date">{{ post.date | date: "%b %Y" }}</div>
<div class="row-title">{{ post.title }}</div>
<div class="row-excerpt">{{ post.excerpt | strip_html | truncatewords: 24 }}</div>
</a>
{% endfor %}
</div>
{% endif %}
{% endfor %}
{% for category in all_categories %}
{% unless ordered contains category %}
<div class="listing-group">
<div class="listing-group-label">{{ category }}</div>
{% for post in sorted_posts %}
{% if post.category == category %}
<a class="listing-row" href="{{ post.url }}">
<div class="row-date">{{ post.date | date: "%b %Y" }}</div>
<div class="row-title">{{ post.title }}</div>
<div class="row-excerpt">{{ post.excerpt | strip_html | truncatewords: 24 }}</div>
</a>
{% endif %}
{% endfor %}
</div>
{% endunless %}
{% endfor %}
</div>
