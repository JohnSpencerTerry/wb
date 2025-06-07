---
layout: default
title: "Home"
---

# Articles

<ul>
  {% assign sorted_articles = site.articles | sort: 'path' %}
  {% for article in sorted_articles %}
    <li><a href="{{ article.url }}">{{ article.title }}</a></li>
  {% endfor %}
</ul>
<h2>Debug Dump</h2>
<pre>
{{ site.articles | inspect }}
</pre>