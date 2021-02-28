---
layout: post
title: "SCons, automated dependency handling"
date: 2021-02-26 8:00:00 +0100
tags: [scons, series, c++]
comments: true
author: Adam Hlavatovic
---

In the previous [post]({% post_url 2021-01-27-scons-checking-dependencies %}) we introduced library dependency checking based on `pkg-config --exists LIBRARY` command. In this post we go even further and let our script generate also `pkg-config` based list of cflags and library dependencies from `dependencies` variable.

First lets change definition of `dependencies` variable from simple list of library names to list of `(library, version)` pairs or simple library name as string (same as before). So we can now have `deendencies` defined this way

```python
dependencies = [
	('glm', '>= 0.9.9.5'),
	('bullet', '>= 2.88'),
	'OGRE-Bites',
]
```
where we want to check existance of `glm >= 0.9.9.5`, `bullet >= 2.88` libraries and we only care about any `OGRE-Bites` library version is availaable.

The second change, `configure()` function defined this way

```python
def configure(env, dependency_list):
	conf = env.Configure(
		custom_tests={'CheckPkgVersion': check_pkg_version})

	for dep in dependency_list:
		pkg = ("%s %s" % dep) if type(dep) == tuple else dep
		if not conf.CheckPkgVersion(pkg):
			print("'%s' library not found" % pkg)
			Exit(1)

	conf_env = conf.Finish()

	pkg_conf = 'pkg-config --cflags --libs ' + ' '.join(  # 1
		map(lambda dep: dep[0] if type(dep) == tuple else dep, dependencies))

	conf_env.ParseConfig(pkg_conf)

	return conf_env
```

which returns `dependencies` based auto configured build environment variable. The function check dependencies the same way as before (in `build()` function), but this time it also create `pkg-config` based configuration string and calls `ParseConfig()` environment function (1). In a case of our sample `pkg-config` generated configuration string looks this way

```
pkg-config --cflags --libs glm >= 0.9.9.5 bullet >= 2.88 OGRE-Bites
```

Function `configure()` is ment to be used this way

```python
dependencies = [
	('glm', '>= 0.9.9.5'),
	('bullet', '>= 2.88'),
	'OGRE-Bites',
]

env = Environment()
env = configure(env, dependencies)

# ...
```

Full `SConstruct` sample file listing there

```python
# automated pkg-config based dependencies with version check

# List of pkg-config based library dependencies as (library, version) pair or just package as string (e.g. ('libzmq', '>= 4.3.0') pair or 'libzmq' string).
dependencies = [
	('glm', '>= 0.9.9.5')
]

def build():
	cpp17 = Environment(CXXFLAGS=['-std=c++17'], CCFLAGS=['-Wall', '-g', '-O0'])

	cpp17 = configure(cpp17, dependencies)

	cpp17.Program('main.cpp')


def configure(env, dependency_list):
	conf = env.Configure(
		custom_tests={'CheckPkgVersion': check_pkg_version})

	for dep in dependency_list:
		pkg = ("%s %s" % dep) if type(dep) == tuple else dep
		if not conf.CheckPkgVersion(pkg):
			print("'%s' library not found" % pkg)
			Exit(1)

	conf_env = conf.Finish()

	pkg_conf = 'pkg-config --cflags --libs ' + ' '.join(
		map(lambda dep: dep[0] if type(dep) == tuple else dep, dependencies))

	conf_env.ParseConfig(pkg_conf)

	return conf_env

def check_pkg_version(context, pkg):
	"""custom pkg-config based package version check"""

	context.Message("Checking for '%s' library ... " % pkg)
	res = context.TryAction("pkg-config --exists '%s'" % pkg)[0]
	context.Result(res)
	return res

build()
```

There is also [configure with dependencies](https://github.com/sansajn/test/tree/master/scons/configure_with_dependencies) github repository sample.
