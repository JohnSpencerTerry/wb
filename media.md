---
layout: default
title: "Reading & Watching"
permalink: /media/
nav: media
---

{% include page-heading.html title=page.title %}
<p>Books, movies, and the occasional TV show or exhibit — a non-exhaustive log of what I've consumed and when. Most entries are just a title; a few link out to something longer I wrote about them.</p>
<div class="media-filter">
  <button type="button" class="media-filter-btn active" data-filter="all">All</button>
  <button type="button" class="media-filter-btn" data-filter="fiction">Fiction</button>
  <button type="button" class="media-filter-btn" data-filter="nonfiction">Nonfiction</button>
  <button type="button" class="media-filter-btn" data-filter="cookbook">Cookbook</button>
  <button type="button" class="media-filter-btn" data-filter="movie">Movie</button>
</div>
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
<a class="media-row media-row-linked" href="{{ entry.article }}" data-genre="{{ entry.genre }}">
{% else %}
<div class="media-row" data-genre="{{ entry.genre }}">
{% endif %}
  <span class="media-date">{{ entry.date | date: "%b %-d" }}</span>
  <span class="media-genre media-genre-{{ entry.genre }}">{{ entry.genre }}</span>
  <span class="media-title">{{ entry.title }}{% if entry.creator %}<span class="media-creator"> — {{ entry.creator }}</span>{% endif %}</span>
{% if entry.article %}
</a>
{% else %}
</div>
{% endif %}
{% endfor %}
</div>
</div>

<script>
(function () {
  var buttons = document.querySelectorAll('.media-filter-btn');
  var rows = document.querySelectorAll('.media-row');
  var groups = document.querySelectorAll('.media-year-group');

  function applyFilter(genre) {
    rows.forEach(function (row) {
      row.hidden = genre !== 'all' && row.dataset.genre !== genre;
    });
    groups.forEach(function (group) {
      var hasVisible = Array.prototype.some.call(
        group.querySelectorAll('.media-row'),
        function (row) { return !row.hidden; }
      );
      group.hidden = !hasVisible;
      var label = group.previousElementSibling;
      if (label && label.classList.contains('media-year')) {
        label.hidden = !hasVisible;
      }
    });
  }

  buttons.forEach(function (btn) {
    btn.addEventListener('click', function () {
      buttons.forEach(function (b) { b.classList.remove('active'); });
      btn.classList.add('active');
      applyFilter(btn.dataset.filter);
    });
  });
})();
</script>
