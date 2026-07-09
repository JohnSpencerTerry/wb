---
layout: default
title: "Life"
permalink: /life/
nav: life
---

{% include page-heading.html title=page.title %}
<p>Trips, time outdoors with our dog, running, and other things away from the keyboard.</p>
{% assign posts = site.articles | where_exp: "a", "a.draft != true" | where: "category", "Life" | sort: 'date' | reverse %}
<div class="listing">
{% for post in posts %}
<a class="listing-row" href="{{ post.url }}">
<div class="row-date">{{ post.date | date: "%b %Y" }}</div>
<div class="row-title">{{ post.title }}</div>
<div class="row-excerpt">{{ post.excerpt | strip_html | truncatewords: 24 }}</div>
</a>
{% endfor %}
</div>
