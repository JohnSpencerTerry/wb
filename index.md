---
layout: default
permalink: /
sitemap: false
---

{% assign writing = site.writing | where_exp: "p", "p.draft != true" | sort: 'date' | reverse %}
{% if writing.size > 0 %}
<div class="home-section">
<div class="section-head"><h2>Latest Write Ups</h2><a class="all-link" href="/writing/">All &rarr;</a></div>
<ul>
{% for post in writing limit: 3 %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
</div>
{% endif %}
{% assign food = site.food | where_exp: "p", "p.draft != true" | sort: 'date' | reverse %}
{% if food.size > 0 %}
<div class="home-section">
<div class="section-head"><h2>From the Kitchen</h2><a class="all-link" href="/food/">All &rarr;</a></div>
<ul>
{% for post in food limit: 2 %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
</div>
{% endif %}
