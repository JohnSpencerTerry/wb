---
layout: default
title: "Food"
permalink: /food/
---

Recipes my wife and I want to keep, plus notes from cooking, shopping, and eating around New York.

{% assign visible_posts = site.food | where_exp: "p", "p.draft != true" %}
{% assign sorted_posts = visible_posts | sort: 'date' | reverse %}
{% assign ordered = "Recipes,Kitchen Notes,NYC Eats" | split: "," %}
{% assign all_categories = sorted_posts | map: 'category' | uniq %}

{% for category in ordered %}
{% assign in_cat = sorted_posts | where: "category", category %}
{% if in_cat.size > 0 %}
## {{ category }}

{% for post in in_cat %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}
{% endif %}
{% endfor %}

{% for category in all_categories %}
{% unless ordered contains category %}
## {{ category }}

{% for post in sorted_posts %}
{% if post.category == category %}
- [{{ post.title }}]({{ post.url }})
{% endif %}
{% endfor %}
{% endunless %}
{% endfor %}
