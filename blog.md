---
layout: default
title: Writing
permalink: /blog/
description: Notes on search, AI personalization, and backend systems, and the trade-offs behind them.
---

<section class="hero">
  <h1>Writing</h1>
  <p class="tagline">Notes on the parts of a system that explain why it works the way it does, search, AI personalization, backend infrastructure, and the trade-offs underneath.</p>
</section>

{% assign writing_builds = site.builds | where: "show_in_writing", true %}
{% assign writing_items = site.posts | concat: writing_builds | sort: "date" | reverse %}

<ul class="posts-index">
  {% for item in writing_items %}
  <li>
    <div class="post-row">
      <a href="{{ item.url | relative_url }}">{{ item.title }}</a>
      <span class="when">{{ item.date | date: "%b %Y" }}</span>
    </div>
    {% if item.excerpt %}<p class="excerpt">{{ item.excerpt | strip_html | truncatewords: 30 }}</p>{% endif %}
  </li>
  {% else %}
  <p>No posts yet, check back soon.</p>
  {% endfor %}
</ul>
