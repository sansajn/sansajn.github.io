---
layout: post
title: "Safe system handle implementation"
date: 2021-03-16 17:00:00 +0100
tags: [c++, recipe]
comments: true
author: Adam Hlavatovic
---

I've saw handy safe system handle implementation in [The Modern C++ Challenge](https://www.packtpub.com/product/the-modern-c-challenge/9781788993869) (problem 21) book and found it handy in case of working with plain *C* file streams (aka. `FILE *`, `fopen()`, `fclose()`, ...). It was originally written for windows `HANDLE`, but it is easy to use it also with `FILE *`, this way

```c++
#include <iostream>
#include <cstdio>
using std::cout;

int main(int argc, char * argv[]) {
	file_handle fin{fopen("system_handle.cpp", "r")};
	if (!fin)
		return 1;

	// read file
	char buf[4096];
	size_t bytes = fread(buf, sizeof(char), 4096, fin.get());
	cout << "readed "<< bytes << " bytes\n";

	cout << "done!\n";
	return 0;
}
```

where `file_handle` is defined as `unique_handle<>` specialization this way

```c++
//! safe system handle wrapper
template <typename Traits>
class unique_handle {
	using pointer = typename Traits::pointer;

public:
	unique_handle(unique_handle const &) = delete;
	unique_handle & operator=(unique_handle const &) = delete;

	explicit unique_handle(pointer value = Traits::invalid()) noexcept
		: _value{value}
	{}

	unique_handle(unique_handle && other) noexcept
		: _value{other.release()}
	{}

	unique_handle & operator=(unique_handle && other) noexcept {
		if (this != &other)
			reset(other.release());
		return *this;
	}

	~unique_handle() noexcept {
		Traits::close(_value);
	}

	explicit operator bool() const noexcept {
		return _value != Traits::invalid();
	}

	pointer get() const noexcept {
		return _value;
	}

	pointer release() noexcept	{
		auto value = _value;
		_value = Traits::invalid();
		return value;
	}

	bool reset(pointer value = Traits::invalid()) noexcept {
		if (_value != value) {
			Traits::close(_value);
			_value = value;
		}
		return static_cast<bool>(*this);
	}

	void swap(unique_handle<Traits> & other) noexcept {
		std::swap(_value, other._value);
	}

private:
	pointer _value;
};

template <typename Traits>
void swap(unique_handle<Traits> & left, unique_handle<Traits> & right) {
	left.swap(right);
}

template <typename Traits>
bool operator==(unique_handle<Traits> const & left, unique_handle<Traits> const & right) noexcept {
	return left.get() == right.get();
}

template <typename Traits>
bool operator!=(unique_handle<Traits> const & left, unique_handle<Traits> const & right) noexcept {
	return left.get() != right.get();
}

struct file_handle_traits {
	using pointer = FILE *;
	static pointer invalid() noexcept {return nullptr;}
	static void close(pointer value) noexcept {fclose(value);}
};

using file_handle = unique_handle<file_handle_traits>;
```

> see [`system_handle.cpp`](https://github.com/sansajn/test/blob/master/c%2B%2B/system_handle.cpp) for full blown sample


At first, I was thinking that thanks to *class template argument deduction* from *C++17* we can use `unique_ptr` with custom deleter to handle `FILE *` this way

```c++
unique_ptr fin{fopen("system_handle.cpp", "r"), &fclose};
```

but it ends up with `error: class template argument deduction failed` comlpain while building. Later, I've found that `unique_ptr` needs to be used this way

```c++
void close_file(FILE * f) {fclose(f);}

void foo() {
	unique_ptr<FILE, decltype(&close_file)> fin{fopen("demo.txt", "r"), &close_file};
	// ...
}
```

to get it working with `FILE *`, which is far from the ideal. If you know the better way just leave me a comment.
