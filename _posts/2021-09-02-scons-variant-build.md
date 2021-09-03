---
layout: post
title: "Variant directory build in SCons"
date: 2021-09-02 8:00:00 +0100
tags: [scons, series, c++]
comments: true
author: Adam Hlavatovic
---

In the previous posts we always build into the current directory, which is the easiest way to go. SCons is aware of all build fragments (`.a`, `.o` and binary files) so clean up can be done automatically with `scons -c` command.

However, sometimes can be useful to build to the custom directory to separate different builds for example debug, release or coverage.

> **tip**: to know more about coverage builds, see [SCons, test coverage report generation]({% post_url 2021-03-17-coverage-report %})

In *SCons* we can use [`VariantDir`][VariantDir] function for that purpose this way

```python
# the simpliest variant directory setup
VariantDir('build', 'source', duplicate=0)
env = Environment()
env.Program('build/main.cpp')
```

where the resulting binary and also object file will be stored in `build` instead of `source` directory. Run

```console
$ scons -Q
g++ -o build/main.o -c source/main.cpp
g++ -o build/main build/main.o
```

to build the sample. See [`test/scons/variant_dir`](https://github.com/sansajn/test/tree/master/scons/variant_dir) for full featured sample.


We can go even further and combine knowledge from previous [Release/debug builds in SCons]({% post_url 2021-07-26-scons-release-debug %}) post and create build script building either to *build-release* or *build-debug* directory based on configuration option. This way

```python
# SConstruct: use scons --build-release in case of optimized binary required

AddOption('--build-release', action='store_true', dest='build_release', 
	help='generate optimized binary', default=False)

build_path = 'build-debug'
if GetOption('build_release'):
	build_path = 'build-release'

VariantDir(build_path, 'source', duplicate=0)

env = Environment(CCFLAGS=['-Wall'])

# apply debug/release options
if not GetOption('build_release'):
	env.Append(CCFLAGS=['-ggdb3', '-Og', '-DDEBUG'])
else: # --build-release
	env.Append(CCFLAGS=['-Os', '-DNDEBUG'])

env.Program('%s/%s' % (build_path, 'main.cpp'))
```

By default *main* is build with a debug settings into `build-debug` directory

```console
$ scons -Q
g++ -o build-debug/main.o -c -Wall -ggdb3 -Og -DDEBUG source/main.cpp
g++ -o build-debug/main build-debug/main.o
```

and with `--build-release` option we can trigger *release* build into `build-release` directory

```console
$ scons --build-release -Q
g++ -o build-release/main.o -c -Wall -Os -DNDEBUG source/main.cpp
g++ -o build-release/main build-release/main.o
```

See [`test/scons/variant-build`](https://github.com/sansajn/test/tree/master/scons/variant-build) for full featured sample.

And that is all for today.


[VariantDir]: https://scons.org/doc/1.2.0/HTML/scons-user/x3346.html
