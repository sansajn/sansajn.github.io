---
layout: post
title: "Code samples from file"
date: 2020-12-21 12:35:00 +0100
categories: jekyll
author: Adam Hlavatovic
---

To insert your code samples from file (stored on github.org or anywhere else) we can use [*remote file content*](https://github.com/dimitri-koenig/jekyll-plugins) jekyll plugin. 

Installation is easy, just copy [`remote_file_content.rb`](http://www.test.com) file to `_plugins` directory in your jekyll powered project.

> **note**: if `_plugins` directory doesn't exists, create it

In your post, write 

{% raw %}
{% highlight cpp %}
{% remote_file_content https://raw.githubusercontent.com/sansajn/cube_rain/master/cube_rain/flat_shader.hpp %}
{% endhighlight %}
{% endraw %}

to insert *C++* code sample from [`flat_shader.hpp`](https://raw.githubusercontent.com/sansajn/cube_rain/master/cube_rain/flat_shader.hpp) file, which results in 

{% highlight cpp %}
{% remote_file_content https://raw.githubusercontent.com/sansajn/cube_rain/master/cube_rain/flat_shader.hpp %}
{% endhighlight %}
