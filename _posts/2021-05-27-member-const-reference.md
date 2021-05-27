---
layout: post
title: "Constant reference member variable"
date: 2021-05-27 08:00:00 +0100
tags: [c++, c++11]
comments: true
author: Adam Hlavatovic
---

Assume following structure `foo` with constant reference as member variable

```c++
stuct foo {
	foo(string const & s) : _s{s} {}
	bool empty() const {return _s.empty();}

	string const & _s;
};
```

which is perfectly working in case of following (1) and (2) usages

```c++
string const s = "hello!";
foo g{s};
assert(!g.empty());  // (1) OK, working as expected

string s_mut = "hello!";
foo h{s_mut};
assert(!h.empty());  // (2) OK, working as expected
```

but not working as expected in following (3) use case

```c++
foo f{"hello!"};
assert(!f.empty());  // (3) Wrong, not working as expected
```

Usage (3) is actually not working at all, but before we dive into what is wrong there let me describe one interesting C++ feature. We are talking there about extending temporary variable lifetime by binding to a constant reference, this way

```c++
string temporary() {return "hello!";}

void goo() {
	string const & s = temporary();  // s refer to valid string objects
}
```

I wanted to use above C++ feature in my `foo` structure implementation, but it turns out that this only works in case of local variables.

Now, get back to our `foo` implementation. The think there in usage (3) is that our temporary string object in constructor call `foo f{"hello!"}` is bind to a constant reference argument variable `s`. Even we store `s` to `_s` member variable in the constructor, temporary string object is destroyed at the end of the constructor call and we end up with dangling reference stored as `_s`.

Ok, we now know what is wrong there, but how we can fix it?

It turns out that the best think we can do is to make usage (3) impossible by deleting rvalue reference constructor this way

```c++
struct foo {
	foo(string &&) = delete;
	// the rest is the same as before ...
};
```

That it all for today, if you want know more about extending temporaries lifetime see [Important const”](https://herbsutter.com/2008/01/01/gotw-88-a-candidate-for-the-most-important-const/) article from Herb Sutter.
