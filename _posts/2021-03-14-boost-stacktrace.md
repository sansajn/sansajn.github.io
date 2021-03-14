---
layout: post
title: "Better crash diagnostic with stacktrace"
date: 2021-03-14 13:00:00 +0100
tags: [boost, c++, debug]
comments: true
author: Adam Hlavatovic
---

Have you ever seen 

```console
$ ./hello 
Segmentation fault (core dumped)
```

in your program?

Wouldn't be great to see 

```console
$ ./hello 
signal 'Segmentation fault' (11) caught
stacktrace:
 0# verbose_signal_handler(int) at /home/ja/devel/test/boost/stacktrace/hello.cpp:12
 1# 0x00007FB3CE555950 in /lib/x86_64-linux-gnu/libc.so.6
 2# gsignal in /lib/x86_64-linux-gnu/libc.so.6
 3# foo::seg_fault() at /home/ja/devel/test/boost/stacktrace/hello.cpp:21
 4# goo() at /home/ja/devel/test/boost/stacktrace/hello.cpp:27
 5# main at /home/ja/devel/test/boost/stacktrace/hello.cpp:32
 6# __libc_start_main in /lib/x86_64-linux-gnu/libc.so.6
 7# _start in ./hello
```

instead of previous output? If both of you answers was **YES**, then continue reading, because in the post I will show how it can be done with `boost::stacktrace` (as always the easy way!).

Actually, with `boost::stacktrace` having stacktrace on a signal it is pretty easy, the hardest part is to know that `boost::stacktrace` even exist.

## Signal handler setup

The first step, setup signal handler with `std::signal()` function

```c++
void verbose_signal_handler(int signal) {
	// ...
}

int main() {
	signal(SIGSEGV, verbose_signal_handler);
	// ...
}
```

in our case we setuped handler for SIGSEGV (Segmentation fault) signal. You know when you are trying to write/read memory you don't own, like this

```c++
int * p = nullptr;
// a lot of lines of code ...
*p = 42;  // (1)
```

where we tried wrote to `0x0` memory address (1).

> **tip**: posix defines more feature rich [`sigaction()`](https://man7.org/linux/man-pages/man2/sigaction.2.html) function to setup custom signal handler  


## Generate stacktrace output

The second step is to create `boost::stacktrace::stacktrace` instance and use it with stream object (like `cout`), this way

```c++
void verbose_signal_handler(int signal) {
	cout << "signal '" << strsignal(signal) << "' (" << signal << ") caught\n"
		<< "stacktrace:\n"
		<< boost::stacktrace::stacktrace{} 
		<< endl;

	exit(signal);
}
```

There it is full blown sample

```c++
// boost stackstrace sample
#include <iostream>
#include <csignal>
#include <boost/stacktrace.hpp>
#include <string.h>  // for posix strsignal()

using std::cout, std::endl;

void verbose_signal_handler(int signal) {
	cout << "signal '" << strsignal(signal) << "' (" << signal << ") caught\n"
		<< "stacktrace:\n"
		<< boost::stacktrace::stacktrace{} 
		<< endl;

	exit(signal);
}

struct foo {
	void seg_fault() {
		raise(SIGSEGV);
	}
};

void goo() {
	foo f;
	f.seg_fault();
}

int main(int argc, char * argv[]) {
	signal(SIGSEGV, verbose_signal_handler);
	goo();
	cout << "done!\n";
	return 0;
}
```

## Linker setup

To see function names instead of raw addresses and file names with line instead of program binary name, `boost::stacktrace` needs little bit of help from linker and other libraries.

The sample was build this way

```console
$ scons hello
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
g++ -o hello.o -c -std=c++17 -Wall -g -O0 -DBOOST_STACKTRACE_USE_BACKTRACE hello.cpp
g++ -o hello -rdynamic hello.o -lboost_stacktrace_backtrace -lboost_system -lboost_filesystem -ldl
scons: done building targets.
```

> see [`hello.cpp`](https://github.com/sansajn/test/blob/master/boost/stacktrace/hello.cpp) and [`SConstruct`](https://github.com/sansajn/test/blob/master/boost/stacktrace/SConstruct) build script for further informations

and as you can see you need to define `BOOST_STACKTRACE_USE_BACKTRACE` and link program with `lboost_stacktrace_backtrace` and `dl` libraries and add all symbols with `-rdynamic` linker option.

> **-rdynamic**: Pass the flag -export-dynamic to the ELF linker, on targets that support it. This instructs the linker to add all symbols, not only used ones, to the dynamic symbol table. This option is needed for some uses of dlopen or to allow obtaining backtraces from within a program. 
