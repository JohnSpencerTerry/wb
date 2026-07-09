---
layout: default
title: "Life"
permalink: /life/
---

Trips, time outdoors with our dog, running, and other things away from the keyboard.

{% assign posts = site.articles | where_exp: "a", "a.draft != true" | where: "category", "Life" | sort: 'date' | reverse %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}
