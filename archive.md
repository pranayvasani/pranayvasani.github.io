---
layout: default
title: Archive
permalink: /archive
---

<h1 class="page-title">Archive</h1>
<p class="page-subtitle">{{ site.posts | size }} posts since 2020</p>

{% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
{% for year_group in posts_by_year %}
<section class="archive-year">
  <h2 class="archive-year__heading">{{ year_group.name }}</h2>
  <ul class="archive-year__posts">
    {% for post in year_group.items %}
    <li class="archive-post">
      <time class="archive-post__date">{{ post.date | date: "%b %d" }}</time>
      <a href="{{ post.url | relative_url }}" class="archive-post__title">{{ post.title | escape }}</a>
      {% if post.tags and post.tags.size > 0 %}
      <span class="archive-post__tags">
        {% for tag in post.tags %}<span class="tag-badge tag-badge--small">{{ tag }}</span>{% endfor %}
      </span>
      {% endif %}
    </li>
    {% endfor %}
  </ul>
</section>
{% endfor %}
