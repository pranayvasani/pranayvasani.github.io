---
layout: default
title: Topics
permalink: /topics
---

<h1 class="page-title">Topics</h1>

{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}
<section class="topic-section" id="{{ tag[0] | downcase | replace: ' ', '-' }}">
  <h2 class="topic-heading">
    {{ tag[0] }}
    <span class="topic-count">{{ tag[1] | size }}</span>
  </h2>
  <ul class="topic-posts">
    {% assign topic_posts = tag[1] | sort: "date" | reverse %}
    {% for post in topic_posts %}
    <li class="topic-post">
      <a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
      <span class="topic-post-date">{{ post.date | date: "%b %Y" }}</span>
    </li>
    {% endfor %}
  </ul>
</section>
{% endfor %}
