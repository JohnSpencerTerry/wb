---
layout: default
permalink: /
sitemap: false
---

I'm John, a software engineer in New York City. This site is where I work on my writing.

{% assign articles = site.articles | where_exp: "a", "a.draft != true" | where_exp: "a", "a.category != 'Life'" | where_exp: "a", "a.category != 'Book Reviews'" | sort: 'date' | reverse %}
{% if articles.size > 0 %}
## Latest articles

{% for article in articles limit: 3 %}
- [{{ article.title }}]({{ article.url }})
{% endfor %}

[All articles →](/articles/)
{% endif %}

{% assign food = site.food | where_exp: "p", "p.draft != true" | sort: 'date' | reverse %}
{% if food.size > 0 %}
## From the kitchen

{% for post in food limit: 3 %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

[More food →](/food/)
{% endif %}

{% assign life = site.articles | where_exp: "a", "a.draft != true" | where: "category", "Life" | sort: 'date' | reverse %}
{% if life.size > 0 %}
## Life

{% for post in life limit: 2 %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}

[More life →](/life/)
{% endif %}

{% assign stories = site.creative_writing | where_exp: "s", "s.draft != true" | sort: 'date' | reverse %}
{% if stories.size > 0 %}
## Fiction

{% for story in stories limit: 2 %}
- [{{ story.title }}]({{ story.url }})
{% endfor %}

[More writing →](/creative-writing/)
{% endif %}

{% assign projects = site.side_projects | where_exp: "p", "p.draft != true" | sort: 'date' | reverse %}
{% if projects.size > 0 %}
## Side projects

{% for project in projects limit: 2 %}
- [{{ project.title }}]({{ project.url }})
{% endfor %}

[More projects →](/side-projects/)
{% endif %}
