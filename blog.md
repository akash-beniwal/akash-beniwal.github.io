---
title: Writing
permalink: /blog/
---

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
