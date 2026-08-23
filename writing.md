---
layout: default
title: "Write Ups"
permalink: /writing/
nav: writing
---

{% include page-heading.html title=page.title %}
<p>Articles, side projects, and stories, filterable by tag. Select multiple tags to narrow to posts that match all of them.</p>
<div class="media-filter">
  <button type="button" class="media-filter-btn active" data-filter="all">All</button>
  <button type="button" class="media-filter-btn" data-tag="Tech">Tech</button>
  <button type="button" class="media-filter-btn" data-tag="AI">AI</button>
  <button type="button" class="media-filter-btn" data-tag="Side Project">Side Project</button>
  <button type="button" class="media-filter-btn" data-tag="Fiction">Fiction</button>
  <button type="button" class="media-filter-btn" data-tag="Random">Random</button>
</div>
{% assign visible_posts = site.writing | where_exp: "p", "p.draft != true" | sort: 'date' | reverse %}
<div class="listing">
{% for post in visible_posts %}
<a class="listing-row" href="{{ post.url }}" data-tags="{{ post.tags | join: ',' }}">
<div class="row-date">{{ post.date | date: "%b %Y" }}</div>
<div class="row-title">{{ post.title }}</div>
<div class="row-excerpt">{{ post.excerpt | strip_html | truncatewords: 24 }}</div>
<div class="row-tags">{% for tag in post.tags %}<span class="row-tag">{{ tag }}</span>{% endfor %}</div>
</a>
{% endfor %}
</div>

<script>
(function () {
  var allBtn = document.querySelector('.media-filter-btn[data-filter="all"]');
  var tagButtons = document.querySelectorAll('.media-filter-btn[data-tag]');
  var rows = document.querySelectorAll('.listing-row[data-tags]');

  function activeTags() {
    return Array.prototype.map.call(
      document.querySelectorAll('.media-filter-btn[data-tag].active'),
      function (b) { return b.dataset.tag; }
    );
  }

  function applyFilter() {
    var active = activeTags();
    if (active.length === 0) {
      allBtn.classList.add('active');
    } else {
      allBtn.classList.remove('active');
    }
    rows.forEach(function (row) {
      var rowTags = row.dataset.tags ? row.dataset.tags.split(',') : [];
      var matches = active.every(function (tag) { return rowTags.indexOf(tag) !== -1; });
      row.hidden = !matches;
    });
  }

  allBtn.addEventListener('click', function () {
    tagButtons.forEach(function (b) { b.classList.remove('active'); });
    applyFilter();
  });

  tagButtons.forEach(function (btn) {
    btn.addEventListener('click', function () {
      btn.classList.toggle('active');
      applyFilter();
    });
  });

  applyFilter();
})();
</script>
