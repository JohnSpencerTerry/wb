---
layout: default
title: "What I'm Reading"
permalink: /book-reviews/
nav: books
---

{% include page-heading.html title=page.title %}
<p>Books I've read lately, mostly engineering, occasionally fiction.</p>
{% assign reviews = site.articles | where_exp: "a", "a.draft != true" | where: "category", "Book Reviews" | sort: 'date' | reverse %}
{% if reviews.size > 0 %}
<div class="listing">
{% for review in reviews %}
<a class="listing-row" href="{{ review.url }}">
<div class="row-date">{{ review.date | date: "%b %Y" }}</div>
<div class="row-title">{{ review.title }}</div>
<div class="row-excerpt">{{ review.excerpt | strip_html | truncatewords: 24 }}</div>
</a>
{% endfor %}
</div>
{% else %}
<p class="row-excerpt">No reviews published yet.</p>
{% endif %}
