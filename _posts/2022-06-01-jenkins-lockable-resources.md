---
layout: post
title: "Lockable resources in Jenkins"
date: 2022-04-14 8:00:00 +0100
tags: [jenkins, series, CI/CD]
comments: true
author: Adam Hlavatovic
---

The folowing lines describes lockable resources support in Jenkins. The motivation to use lockable resource is this: Let's say you want to delegate building of your project to third-party build software which can handle only one build at time. To prevent build fails for other projects trying to use the build software during building with the build software Jenkins offer support for lockable resources. Lockable resource can be acquired by running Jenkins job and other jobs trying to acquire the same lockable resource are moved to *Build Queue* where are wayting until lockable resoure is released by running job.

To setup lockable resource in Jenkins go to `Manage Jenkins > Configure System` and there in section *Lockable Resources Manager* fill *Resource* form this way:

Name: build-lock
Description: Third party build software lock.

> **note**: leave *Labels* and *Reserverd by* fields empty

and then click to *Add Lockable Resource* button.

In your project configuration (*Configure* option) in a *General* section check *This build requires lockable resources* option and set *Resources* field to `build-lock`, click *Save* button and we are done.
