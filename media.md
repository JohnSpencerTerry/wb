---
layout: default
title: "Reading & Watching"
permalink: /media/
nav: media
---

{% include page-heading.html title=page.title %}
<p>Books, movies, and the occasional TV show or exhibit — a non-exhaustive log of what I've consumed and when. Most entries are just a title; a few link out to something longer I wrote about them.</p>
{% assign entries = site.data.media | sort: 'date' | reverse %}
{% assign current_year = "" %}
<div class="media-list">
{% for entry in entries %}
{% assign entry_year = entry.date | date: "%Y" %}
{% if entry_year != current_year %}
{% unless current_year == "" %}</div>{% endunless %}
<div class="listing-group-label media-year">{{ entry_year }}</div>
<div class="media-year-group">
{% assign current_year = entry_year %}
{% endif %}
{% if entry.article %}
<a class="media-row media-row-linked" href="{{ entry.article }}">
{% else %}
<div class="media-row">
{% endif %}
  <span class="media-date">{{ entry.date | date: "%b %-d" }}</span>
  <span class="media-type media-type-{{ entry.type }}">{{ entry.type }}</span>
  <span class="media-title">{{ entry.title }}{% if entry.creator %}<span class="media-creator"> — {{ entry.creator }}</span>{% endif %}</span>
{% if entry.article %}
</a>
{% else %}
</div>
{% endif %}
{% endfor %}
</div>
</div>
