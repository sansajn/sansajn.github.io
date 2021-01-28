---
layout: post
title: "SCons, checking project dependencies"
date: 2021-01-27 18:50:00 +0100
tags: [scons, series, c++]
comments: true
author: Adam Hlavatovic
---

In the previous [post]({% post_url 2020-12-22-scons-starter %}) in section *third party dependencies* we introduced `pkg-config` command to handle third party library dependancies.

Command `pkg-config` is prefered way in *Linux* not only to provide list of compiler flags (with `--cflags`) or libraries (with `--libs`), but it can be also used to check library version.

For example, let's say we installed `libglm-dev` package (provides OpenGL Mathematics) so we can use

> **tip**: use `sudo apt install libglm-dev` command to install the library

```console
$ pkg-config --list-all|grep -i glm
glm                            GLM - OpenGL Mathematics
```

command to verify `pkg-config` knows about *GLM* library.

> **note**: as shown in previous post we can use `pkg-config --cflags --libs glm` command for a list of compiler flags and libraries (*GLM* is header only library so the command returns `""`)

Then we can use

```console
$ pkg-config --modversion glm
0.9.9.6
```

to see library version. We can also ask `pkg-config` whether specific library is installed with

```bash
pkg-config --exists 'glm >= 0.9.9.0'
```

command.

> **note**: version part of the library name `>= 0.9.9.0` is optional

The command returns true whether *GLM* library version 0.9.9.0 or greater is installed or false otherwise.

*SCons* supports a [*configure context*][configure_context] which can be used to integrate `pkg-config --exists PKG` command this way

```python
dependencies = ['glm >= 0.9.9.5']

def build():
	env = Environment()

	conf = env.Configure(                                    # 1
		custom_tests={'CheckPkgVersion': check_pkg_version})

	for pkg in dependencies:
		if not conf.CheckPkgVersion(pkg):                     # 2
			print("fatal: '%' package not found" % pkg)
			Exit(1)

	env = conf.Finish()

	env.Program('main.cpp')

# custom pkg-config based package version check
def check_pkg_version(context, pkg):
	context.Message("Checking for '%s' package... " % pkg)
	res = context.TryAction("pkg-config --exists '%s'" % pkg)[0]
	context.Result(res)
	return res

build()
```

where `dependencies` variable defines list of dependencies (in our case there is just one dependency there). Line 1 creates [*configure context*][configure_context] as `conf` variable with a custom test function `CheckPkgVersion` so we can later call `conf.CheckPkgVersion(pkg)` on line 2 with a library name (in our case `'glm >= 0.9.9.5'`).

That is not all, [*configure context*][configure_context] comes with a bunch of predifined functions like `CheckCXXHeader()`, `CheckFunc()`, `CheckLib()`, `CheckTypeSize()` and many more. So for example we can check `iostream` header with

```python
if not conf.CheckCXXHeader('iostream'):
	print("We really need 'iostream' header to be installed")
	Exit(1)
```

code. Watch out [SConstruct][configure_sample] file for full futured build script sample and [`main.cpp`][main_sample] for source code sample. Running `scons` command in sample directory produce

```console
$ scons
scons: Reading SConscript files ...
Checking for C++ header file iostream... yes
Checking for 'glm >= 0.9.9.5' package... yes
scons: done reading SConscript files.
scons: Building targets ...
g++ -o main.o -c -std=c++17 -Wall -g -O0 main.cpp
g++ -o main main.o
scons: done building targets.
```

output.

Pretty cool, what do you think?

[configure_context]: https://scons.org/doc/production/HTML/scons-man.html#configure_contexts
[configure_sample]: https://github.com/sansajn/test/blob/master/scons/configure/SConstruct
[main_sample]: https://github.com/sansajn/test/blob/master/scons/configure/main.cpp
