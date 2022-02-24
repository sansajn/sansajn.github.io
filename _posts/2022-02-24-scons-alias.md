---
layout: post
title: "Target aliases"
date: 2022-02-24 8:00:00 +0100
tags: [scons, series, c++]
comments: true
author: Adam Hlavatovic
---

In the previous post from the *SCons* series we were talking about [variant builds]({% post_url 2021-09-02-scons-variant-build %}), today we are going to talk about *target aliases*. 

Let's say you developed an application with unit tests and you wrote following simple build script 

```python
cpp20 = Environment(
	CXXFLAGS=['-std=c++20'],
	CCFLAGS=['-Wall', '-Wextra'])

cpp20.Program('main.cpp')
cpp20.Program('catch_test.cpp')
```

for building it.

With above build script we can execute `scons` command to build both the application as `main` and unit tests as `catch_test` binaries. Hovewer that is not end of the story, we can also execute `scons main` to build only the application or `scons catch_test` to build unit tests. Cool, isn't it? 

With *target aliases* we cen go further, let's say we want execute `scons app` to build the application and `scons test` to build unit tests. With`Environment.Alias()` function it is easy to do that this way

```python
cpp20 = Environment(
	CXXFLAGS=['-std=c++20'],
	CCFLAGS=['-Wall', '-Wextra'])

app = cpp20.Program('main.cpp')
test = cpp20.Program('catch_test.cpp')

cpp20.Alias('app', app)   # 1
cpp20.Alias('test', test) # 2
```

Where line 1 sets alias `app` for building the application and line 2 sets `test` alias for building unit tests. 

Described *target aliases* example can be found in [`test/scons/alias`](https://github.com/sansajn/test/tree/master/scons/alias) repository.


I have one more cool *target aliases* use case. I'm reading a book with C++ samples structured into chapters and I used *target aliases* to group samples for each chapter. I can run `scons ch8` to build samples from chapter 8 or `scons ch10` for  chapter 10 samples this way

```python
# chapter 8 samples
ch8 = [
	cpp20.Program('chapter8/a.cpp'),
	cpp20.Program('chapter8/b.cpp')
]

cpp20.Alias('ch8', ch8)

# chapter 10 samples
ch10 = [
	cpp20.Program('chapter10/a.cpp'),
	cpp20.Program('chapter10/b.cpp')
]

cpp20.Alias('ch10', ch10)
```

And that is all for today.
