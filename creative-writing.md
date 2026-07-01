---
layout: default
title: "Creative Writing"
permalink: /creative-writing/
---

{% assign visible_stories = site.creative_writing | where_exp: "s", "s.draft != true" %}
{% assign sorted_stories = visible_stories | sort: 'date' | reverse %}

{% for story in sorted_stories %}
- [{{ story.title }}]({{ story.url }})
{% endfor %}
