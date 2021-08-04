---
layout: post
title: "Catch2 event listeners"
date: 2021-08-04 8:00:00 +0100
tags: [catch, test, c++]
comments: true
author: Adam Hlavatovic
---

[Catch2](https://github.com/catchorg/Catch2) is great testing library for C++ allows writing tests in C++ way which can be further adjusted to fulfill your neeeds. Today I will talk about *event listeners* in *Catch*, but let's first start with some code sample `event_listener`

```c++
#define CATCH_CONFIG_MAIN
#include <catch.hpp>

TEST_CASE("A") {
	SECTION("A.1") {}
	SECTION("A.2") {}
}

TEST_CASE("B") {
	REQUIRE(true);
}
```

The output of `event_listener` program is

```console
$ ./event_listener 
===============================================================================
All tests passed (1 assertion in 2 test cases)
```

By default *Catch* only creates overall result at the end of testing, but let's say we want *Catch* to print test name at the beginning of each test (for easier debugging).

We can use *event listeners* for that job! All we need to do is derive from `TestEventListenerBase` class and override event functions we are interested (in our case `testCaseStarting` and `sectionStarting`) this way

```c++
namespace Catch {

struct custom_listener : TestEventListenerBase {
	using TestEventListenerBase::TestEventListenerBase;

	void testCaseStarting(TestCaseInfo const & info) override {
		TestEventListenerBase::testCaseStarting(info);
		stream << "test '" << info.name << "'\n";
	}

	void sectionStarting(SectionInfo const & info) override {
		TestEventListenerBase::sectionStarting(info);
		stream << "section '" << info.name << "'\n";
	}
};

CATCH_REGISTER_LISTENER(custom_listener)

}  // Catch
```

which creates output like

```console
$ ./event_listener 
test 'A'
section 'A'
section 'A.1'
section 'A'
section 'A.2'
test 'B'
section 'B'
===============================================================================
All tests passed (1 assertion in 2 test cases)
```

See full [`event_listener.cpp`](https://github.com/sansajn/test/blob/master/catch/event_listener.cpp) sample implementation on *github*.
