---
layout: post
title: "Google Analytics in Jekyll"
date: 2020-12-22 10:30:00 +0100
categories: jekkyl
comments: true
author: Adam Hlavatovic
---

This post contains step by step guide to setup [google analytics](https://analytics.google.com) for your *Jekyll* powered page. The only requirements are google account and working *Jekyll* powered page.

- log into [analytics](https://analytics.google.com) with your google account

- setup analytics account and property in my case *github.io* and *sansajn.github.io* as property

- setup Data Stream for *sansajn.github.io* property from previous step

- copy *Global Site Tag*, in my case

```html
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-88ZTXK100S"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-88ZTXK100S');
</script>
```

> **note**: your measurement id (in my case G-88ZTXK100S) will differ so do not just copy paste from there

- create `google-analytics.html` in `_includes` and paste GST (**G**lobal **S**ite **T**ag) from previous step and save

- copy `default.html` from `/var/lib/gems/2.7.0/gems/minima-2.5.1/_layouts/` to `_layouts` in your *Jekyll* project

- open copied `default.html` and insert

{% highlight html %}
{% raw %}
{% include google-analytics.html %}
{% endraw %}
{% endhighlight %}

line just after `include head.html` liquid statement between opening `<html>` and `<body>` HTML elements, this way

{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html lang="{{ page.lang | default: site.lang | default: "en" }}">

  {%- include head.html -%}
  {% include google-analytics.html %}

  <body>

    {%- include header.html -%}

    <main class="page-content" aria-label="Content">
      <div class="wrapper">
        {{ content }}
      </div>
    </main>

    {%- include footer.html -%}

  </body>

</html>
{% endraw %}
{% endhighlight %}

and you are done!
