---
layout: default
title: "Creative Writing"
permalink: /creative-writing/
nav: creative-writing
---

{% include page-heading.html title=page.title %}
{% assign visible_stories = site.creative_writing | where_exp: "s", "s.draft != true" | sort: 'date' | reverse %}
<div class="listing">
{% for story in visible_stories %}
<a class="listing-row" href="{{ story.url }}">
<div class="row-date">{{ story.date | date: "%b %Y" }}</div>
<div class="row-title">{{ story.title }}</div>
<div class="row-excerpt">{{ story.excerpt | strip_html | truncatewords: 24 }}</div>
</a>
{% endfor %}
</div>
