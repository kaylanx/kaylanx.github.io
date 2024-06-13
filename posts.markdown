---
layout: page
title: Posts
---

<div>
  {% for post in site.posts limit:1 %}
  <h1><a href="{{ post.url | prepend: site.url }}">{{ post.title }}</a></h1>
  <time datetime="{{ post.date | date_to_xmlschema }}" class="post-date">{{ post.date | date: "%B %e, %Y" }}</time>
  <p class="lead">{{ post.lead }}</p>
  {{ post.content }}
  {% endfor %}
</div>

{% include archive.html %}