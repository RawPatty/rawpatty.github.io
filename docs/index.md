---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: custom_home
title: "Home"
---

# You've reached my blog

I'm John Sun, and this is my tech blog where I share my experiences working with Microsoft technologies.

## Latest Posts

{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url | relative_url }}) - {{ post.date | date: "%B %d, %Y" }}
{% endfor %}

[View all post categories]({{ '/categories/' | relative_url }})