---
layout: page
title: Series
permalink: /series/
---

List of series ...

## OGRE guide series

{% assign posts = site.posts|where: "tags", "series"|where: "tags", "ogre"|reverse %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

## SCons series

{% assign posts = site.posts|where: "tags", "series"|where: "tags", "scons"|reverse %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

## C++20 series

{% assign posts = site.posts|where: "tags", "series"|where: "tags", "c++20"|reverse %}
{% for post in posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
