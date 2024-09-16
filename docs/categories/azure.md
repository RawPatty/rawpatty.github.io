---
layout: category
title: "Azure"
category: "Azure"
permalink: /categories/azure/
---

<article class="page-content">

  {% for post in site.categories[page.category] %}
    <div class="post-preview">
      <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
      <p class="post-meta">{{ post.date | date: "%B %d, %Y" }}</p>
      <p>{{ post.excerpt | strip_html | truncatewords: 50 }}</p>
      <a href="{{ post.url | relative_url }}">Read more</a>
    </div>
    <hr>
  {% endfor %}

</article>
