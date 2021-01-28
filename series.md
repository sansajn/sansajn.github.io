---
layout: page
title: Series
permalink: /series/
---

List of series ...

## OGRE guide

{% assign posts = site.posts|where: "tags", "series"|where: "tags", "ogre"|reverse %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

## SCons guide

{% assign posts = site.posts|where: "tags", "series"|where: "tags", "scons"|reverse %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
