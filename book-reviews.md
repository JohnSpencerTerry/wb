---
layout: default
title: "Book Reviews"
permalink: /book-reviews/
---

Short notes on what I'm reading — mostly engineering, occasionally fiction.

{% assign reviews = site.articles | where_exp: "a", "a.draft != true" | where: "category", "Book Reviews" | sort: 'date' | reverse %}
{% for review in reviews %}
- [{{ review.title }}]({{ review.url }})
{% endfor %}
