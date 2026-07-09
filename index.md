---
layout: default
permalink: /
sitemap: false
---

<p class="home-intro">I'm John, a software engineer in New York City. This site is where I work on my writing. I tend to focus on data and product engineering, cooking, and fiction, with the occasional book review or note from a trip.</p>
{% assign articles = site.articles | where_exp: "a", "a.draft != true" | where_exp: "a", "a.category != 'Life'" | where_exp: "a", "a.category != 'Book Reviews'" | sort: 'date' | reverse %}
{% if articles.size > 0 %}
<div class="home-section">
<div class="section-head"><h2>Latest Articles</h2><a class="all-link" href="/articles/">All &rarr;</a></div>
<ul>
{% for article in articles limit: 3 %}
<li><a href="{{ article.url }}">{{ article.title }}</a></li>
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
{% assign life = site.articles | where_exp: "a", "a.draft != true" | where: "category", "Life" | sort: 'date' | reverse %}
{% if life.size > 0 %}
<div class="home-section">
<div class="section-head"><h2>Life</h2><a class="all-link" href="/life/">All &rarr;</a></div>
<ul>
{% for post in life limit: 2 %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
</div>
{% endif %}
{% assign stories = site.creative_writing | where_exp: "s", "s.draft != true" | sort: 'date' | reverse %}
{% if stories.size > 0 %}
<div class="home-section">
<div class="section-head"><h2>Fiction</h2><a class="all-link" href="/creative-writing/">All &rarr;</a></div>
<ul>
{% for story in stories limit: 2 %}
<li><a href="{{ story.url }}">{{ story.title }}</a></li>
{% endfor %}
</ul>
</div>
{% endif %}
{% assign projects = site.side_projects | where_exp: "p", "p.draft != true" | sort: 'date' | reverse %}
{% if projects.size > 0 %}
<div class="home-section">
<div class="section-head"><h2>Side Projects</h2><a class="all-link" href="/side-projects/">All &rarr;</a></div>
<ul>
{% for project in projects limit: 2 %}
<li><a href="{{ project.url }}">{{ project.title }}</a></li>
{% endfor %}
</ul>
</div>
{% endif %}
