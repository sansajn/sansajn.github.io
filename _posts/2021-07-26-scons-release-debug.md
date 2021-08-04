---
layout: post
title: "Release/debug builds in SCons"
date: 2021-07-26 8:00:00 +0100
tags: [scons, series, c++]
comments: true
author: Adam Hlavatovic
---

SCons doesn't have baked-in support for release/debug build so there is more ways to do it. I've found the one based on command line options (e.g. `--release-build`) usefull and easy to implement. It is based on `AddOption()` function with the same use case and notation as standard `ArgumentParser.add_argument()` from [`argparse`](https://docs.python.org/3.4/library/argparse.html) library.

To define `--release-build` commandline argument option all we needto do is call `AddOption()` this way

```python
AddOption('--release-build', action='store_true', dest='release_build', 
	help='generate optimized binary', default=False)
```

at the beginning of the SConstruct file and then based on commandline options modify the build environment the right way

```python
env = Environment(CCFLAGS=['-Wall'])

# apply debug/release options
if not GetOption('release_build'):
	env.Append(CCFLAGS=['-g', '-Og', '-DDEBUG'])
else: # --release-build
	env.Append(CCFLAGS=['-Os'])
```

Whole SConstruct should looks like


```python
# SConstruct: use scons --release-build in case optimized binery required

AddOption('--release-build', action='store_true', dest='release_build', 
	help='generate optimized binary', default=False)

env = Environment(CCFLAGS=['-Wall'])

# apply debug/release options
if not GetOption('release_build'):
	env.Append(CCFLAGS=['-g', '-Og', '-DDEBUG'])
else: # --release-build
	env.Append(CCFLAGS=['-Os'])

env.Program('main.cpp')
```

and test program `main.cpp` can be simpliest hello world like

```c++
#include <iostream>

int main(int argc, char * argv[]) {
	std::cout << "hello!\n";
	return 0;
}
```

With `scons --release-build` command we can produce optimized binary and with `scons` we can produce unoptimized binary with debug symbols. 

And that is pretty much all for today!
