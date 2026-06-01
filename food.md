---
layout: default
title: "Food"
permalink: /food/
---

Recipes my wife and I want to keep, plus stories from cooking, shopping, and eating around New York.

{% assign visible_posts = site.food | where_exp: "p", "p.draft != true" %}
{% assign sorted_posts = visible_posts | sort: 'date' | reverse %}
{% assign categories = sorted_posts | map: 'category' | uniq %}

{% for category in categories %}
## {{ category }}

{% for post in sorted_posts %}
{% if post.category == category %}
- [{{ post.title }}]({{ post.url }}) <span class="badge">Published {{ post.date | date: "%B %-d, %Y" }}</span>
{% endif %}
{% endfor %}
{% endfor %}
