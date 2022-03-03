---
layout: post
title: "Catch2, testing floats"
date: 2022-03-03 8:00:00 +0100
tags: [catch, series, test, c++]
comments: true
author: Adam Hlavatovic
---

In the previous [Catch2][catch2] post we were talking about [event listeners]({% post_url 2022-03-03-catch2-float %}), today let me show how to test float numbers.

There is not much to say about float number testing in [Catch2][catch2], it is straightforward. We can use `Approx()` object for that, this way

```c++
// testing float numbers
#define CATCH_CONFIG_MAIN
#include <catch.hpp>

TEST_CASE("we can use Approx to test floats", "[floats]") {
	double a = 1.0;
	double b = a + std::numeric_limits<double>::epsilon();
	REQUIRE_FALSE(b == a);
	REQUIRE(b == Approx(a));  // 1
	REQUIRE(b == Approx(a).epsilon(0.1));  // 2
}
```

`Approx` has three member functions `epsilon()`, `margin()` and `scale()` and bunch of operartor overloads. Function `epsilon()` can used to customize $\eps$ constant used during number comparsions, see 2 in the example above. 


[catch2]: https://github.com/catchorg/Catch2
