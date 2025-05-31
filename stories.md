---
layout: page
title: Short Stories
permalink: /stories/
---

# Short Stories

{% for post in site.categories['Short-Story'] %}
  <div class="post-preview">
    <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
    <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
    {% if post.excerpt %}
      <p>{{ post.excerpt | strip_html | truncate: 200 }}</p>
    {% endif %}
  </div>
{% endfor %}
