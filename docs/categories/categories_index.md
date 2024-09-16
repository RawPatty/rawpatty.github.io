---
layout: page
title: "All Categories"
permalink: /categories/
---

<h1>All Categories</h1>

<ul>
  {% assign sorted_categories = site.categories | sort %}
  {% for category in sorted_categories %}
    <li>
      <a href="{{ '/categories/' | append: category[0] | downcase | replace: ' ', '-' | relative_url }}/">{{ category[0] }}</a> ({{ category[1].size }} posts)
    </li>
  {% endfor %}
</ul>

