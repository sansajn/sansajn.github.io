---
layout: post
title: "Remove snap support from Ubuntu 20.04 LTS"
date: 2021-03-12 15:00:00 +0100
tags: [linux, snap]
comments: true
author: Adam Hlavatovic
---

The problem with snap packages are that they are bloated because dependencies are included inside the snap package itself. So to install for example Chromium web browser from the snap package you also install couple of gigabytes of dependencies (even the whole *Ubuntu 18.04 LTS* runtime snap package) which is huge waste even with today's hard drives.

Snap support can be in *Ubuntu 20.04 LTS* removed this way

- remove snap itself with

```bash
sudo apt purge snapd
rm -rf ~/snap
sudo rm -rf /snap
```

commands, then

- mark `snapd` as *persona non grata* so it will not be installed by other package as a dependency with

```bash
sudo apt-mark hold snapd 
```

command.
