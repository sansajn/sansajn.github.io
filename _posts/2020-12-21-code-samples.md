---
layout: post
title: "Code samples from file"
date: 2020-12-21 12:35:00 +0100
categories: jekyll
author: Adam Hlavatovic
---

To insert code sample from file we need to copy our sample somewere to `_includes` directory and use `include` and `highlight` liquids to refer it.

For example to show `_includes/source/main.cpp` *C++* file content we can write

{% raw %}
{% highlight cpp %}
{% include source/main.cpp %}
{% endhighlight %}
{% endraw %}

which results in 

{% highlight cpp %}
{% include source/main.cpp %}
{% endhighlight %}
