---
layout: post
title: "SCons, test coverage report generation"
date: 2021-03-17 18:00:00 +0100
tags: [series, scons, c++]
comments: true
author: Adam Hlavatovic
---

In this part of the *SCons* series we will show how to integrate test coverage report generation for C++ codebase in *SCons* build system.

![test coverage report sample](/assets/image/cpp_code_coverage_report.png)

To generate test coverage report we use `gcovr` command from *gcovr* package so the first step is to install the package with 

```console
$ sudo apt install gcovr
```

command. Next we need something to test. We can use [concurrent queue](concurrent_queue.hpp) implementation for that purpose. This is how it's public interface looks like

```c++
template<typename T>
class concurrent_queue
{
public:
	concurrent_queue();
	concurrent_queue(concurrent_queue const & rhs);
	void push(T x);
	void wait_and_pop(T & result);

	template <typename Rep, typename Period>
	bool wait_and_pop(T & result, std::chrono::duration<Rep, Period> const & rel_time);

	std::shared_ptr<T> wait_and_pop();
	bool try_pop(T & result);
	std::shared_ptr<T> try_pop();
	bool empty() const;

	// ...
};
```

Then we ofcourse need some tests so there they are

```c++
#define CATCH_CONFIG_MAIN
#include <catch.hpp>
#include "concurrent_queue.hpp"

TEST_CASE("we can push and pop elements to the queue", 
	"[concurrent_queue]")
{
	concurrent_queue<int> q;
	REQUIRE(q.empty());
	
	q.push(2);
	q.push(5);
	REQUIRE(!q.empty());

	int task;
	REQUIRE(q.try_pop(task));
	REQUIRE(q.try_pop(task));
	REQUIRE(q.empty());
}
```

Now we have everithing we need. To create test coverage report we need build our tests with `-fprofile-arcs` and `-ftest-coverage` compiler options and with `-lgcovr` linker option. 

> **tip**: go [there](https://gcc.gnu.org/onlinedocs/gcc-10.1.0/gcc/Instrumentation-Options.html) for further options explanation

*SCons* build script looks this way

```python
# generate test coverage
AddOption('--test-coverage', action='store_true', dest='test_coverage', help='enable test coverage analyze (with gcov)', default=False)

env = Environment(CCFLAGS=['-g', '-O0', '-Wall'])

if GetOption('test_coverage'):
	# https://gcc.gnu.org/onlinedocs/gcc-10.1.0/gcc/Instrumentation-Options.html
	env.Append(CXXFLAGS = ['-fprofile-arcs', '-ftest-coverage'],
		LIBS = ['gcov'])

env.Program('test.cpp')
```

where `AddOption()` call defines optional `--test-coverage` option in which case running `test` will produce test coverage analyze input as `test.gcno` file. The next thing we need is call `gcovr` to produce human readable test coverage report with this command

```bash
gcovr -r . --html-details -o coverage_report/index.html
```

which creates test coverage report as `coverage_report/index.html`.

We would like to automate as much things as possible so lets create bash script for that as `coverage` with this content

```bash
#!/bin/bash

COVERAGE_REPORT_PATH=coverage_report

mkdir -p ${COVERAGE_REPORT_PATH}

# run tests
./test

# then create human readable HTML report
gcovr -r . --html-details -o ${COVERAGE_REPORT_PATH}/index.html

# clean up
find  . -depth -type f -name '*.gcda' -exec rm '{}' ';'
```

Now all we need to do is

- build tests with

```bash
scons --test-coverage
```

command and

- run `coverage` script with

```bash
./coverage
```

command.

See [`test_coverage`](https://github.com/sansajn/test/tree/master/scons/test_coverage) repository for full blown code, build file and script sample.
