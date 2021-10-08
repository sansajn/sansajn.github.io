---
layout: post
title: "Exception with stacktrace dump"
date: 2021-10-08 8:00:00 +0100
tags: [c++, exception, boost, debugging]
comments: true
author: Adam Hlavatovic
---

I'm working with a hierarchy based code base these days, where the common mistake is to call default implementation which is meant to be override. Let's jump to the example

```c++
struct base {
	virtual void foo() {}
};

struct impl : public base {};

int main() {
	impl i;
	i.foo();  // calls base::foo()
}
```

where `impl` implementation is expected to reimplement `foo()` member function. Of course, we can define `foo()` as an abstract function, but this is tedious in a case like this

```c++
struct base {
	virtual void a() = 0;
	virtual void b() = 0;
	virtual void c() = 0;
};

struct impl : public base {
	// now we need to override a, b and c even b and c does nothing
	void a() override {/* does something usefull */}
	void b() override {}
	void c() override {}
};
```

where we only need to override one or two member function in a implementation class `impl` (the rest should have an empty implementation).

To avoid to much boilerplate code the code base choosed not to define base member function as abstract functions. As a result, code always compile even expected member function was not defined in an implementation. Most of the time it is time consuming to figure out what is wrong there (especially if the hierarchy is deep enough). I've found that is pretty handy throw an exception in case default member function implementation is called instead of override one, this way

```c++
struct base {
	virtual void foo() {
		throw std::runtime_error{"default implementation, not meant to be called"};
	}
};
```

It is an improvement, but when `base::foo()` is called it produce following output

```console
$ ./test 
terminate called after throwing an instance of 'std::runtime_error'
  what():  default implementation, not meant to be called
```

which doesn't provide clue where in the code default member function implementaion was called.

In case you read one of my previous post [Catching exceptions with GDB]({% post_url 2021-05-28-gdb-catching-exceptions %}), you already know that we can use GDB to see where the call happend, but that is time consuming! 

Wouldn't be cool to have stacktrace dump as part of the exception `what()` message? With [Boost.Stacktrace](https://www.boost.org/doc/libs/master/doc/html/stacktrace.html) library it is easy to do, this way

```c++
#include <stdexcept>
#include <string>
#include <sstream>
#include <boost/stacktrace.hpp>

using std::string, std::exception, std::ostringstream;

class custom_error : public exception {
public:
	custom_error(string const & what) {
		ostringstream oss;
		oss << what << "\n"
			<< "\n"
			<< "Stack trace:\n"
			<< boost::stacktrace::stacktrace{} << "\n";
		
		_err = oss.str();
	}
	
	char const * what() const noexcept override {
		return _err.c_str();
	}

private:
	string _err;
};


int main(int argc, char * argv[]) {
	throw custom_error{"the error message"};
	return 0;
}
```

When the program run it produce following output

```console
$ ./exception_with_stacktrace 
terminate called after throwing an instance of 'custom_error'
  what():  the error message

Stack trace:
 0# custom_error::custom_error(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&) at /home/ja/devel/test/boost/stacktrace/exception_with_stacktrace.cpp:16
 1# main at /home/ja/devel/test/boost/stacktrace/exception_with_stacktrace.cpp:31
 2# __libc_start_main in /lib/x86_64-linux-gnu/libc.so.6
 3# _start in ./exception_with_stacktrace
```

where we can clearly see that throw happens in `main()` function on line 31, see [`exception_with_stacktrace.cpp`](https://github.com/sansajn/test/blob/master/boost/stacktrace/exception_with_stacktrace.cpp) sample and [`SConstruct`](https://github.com/sansajn/test/blob/master/boost/stacktrace/SConstruct) build script to get sample working.

Also read [Better crash diagnostic with stacktrace]({% post_url 2021-03-14-boost-stacktrace %}) post to get more familiar with *Boost.Stacktrace*.
