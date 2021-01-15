---
layout: post
title: "Producing animated images"
date: 2021-01-15 16:31:00 +0100
categories: utils
comments: true
author: Adam Hlavatovic
---

Let's say we have following four images

![head 1](/assets/image/head_animation/head_1.png) ![head 2](/assets/image/head_animation/head_2.png)
![head 3](/assets/image/head_animation/head_3.png) ![head 4](/assets/image/head_animation/head_4.png)

and want to create animated one. Command

```bash
ffmpeg -framerate 4 -i head_%d.png -loop 0 head.webp
```

can be use to produce lossy animated image in WEBP format supported by mayor browsers like Firefox and Chrome. The command produce

![animated head](/assets/image/head_animation/head.webp)

> **tip**: we can create animated image also from video input, in this case just use `-i video.mkv`

WEBP related *ffmpeg* options can be found in [9.10 libwebp](https://ffmpeg.org/ffmpeg-codecs.html#libwebp) ffmpeg's documentation.
