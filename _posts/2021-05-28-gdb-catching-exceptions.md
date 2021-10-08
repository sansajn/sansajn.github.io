---
layout: post
title: "Catching exceptions with GDB"
date: 2021-05-28 08:00:00 +0100
tags: [c++, gdb, debugging ]
comments: true
author: Adam Hlavatovic
---

If your program ever terminates with an exception and you have no idea where to start investigate, then the article is definitely for you. [GDB][gdb] has nice feature allow you to find exception source by using `catch throw` command.

Consider following test program `main.cpp`

```c++
#include <stdexcept>

void unhandled_exception() {
	throw std::runtime_error{"something bad happened there"};
}

int main() {
	unhandled_exception();
	return 0;
}
```

which can be compiled by `g++ -g -O0 main.cpp` command. When we try to run it (of course after building with above command)

```console
$ ./a.out
terminate called after throwing an instance of 'std::runtime_error'
  what():  something bad happened there
Aborted (core dumped)
```

we can see it ends up with *something bad happened there* complains, terminated by unhandled `std::runtime_error` exception.

So how we can find out the source of the exception, the `throw` instruction?

Just load our sample program inside [GDB][gdb] with

```console
$ gdb ./a.out
GNU gdb (Ubuntu 9.2-0ubuntu1~20.04) 9.2
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./a.out...
```

command and then use `catch throw` command this way

```console
(gdb) catch throw
Catchpoint 1 (throw)
```

then run program with

```console
(gdb) start
Temporary breakpoint 2 at 0x1223: file main.cpp, line 7.
Starting program: /home/ja/temp/exception_catching/a.out 

Temporary breakpoint 2, main () at main.cpp:7
7	int main() {
```

command. As you can see GDB stops by default at `main()` function so continue execution with `c[ontinue]` command, this way

```console
(gdb) c
Continuing.

Catchpoint 1 (exception thrown), 0x00007ffff7e64762 in __cxa_throw ()
   from /lib/x86_64-linux-gnu/libstdc++.so.6
```

and there it is, program stoped at *Catchpoint 1 (exception thrown)*. We can now use `where` command to show callstack, this way

```console
(gdb) where
#0  0x00007ffff7e64762 in __cxa_throw () from /lib/x86_64-linux-gnu/libstdc++.so.6
#1  0x0000555555555209 in unhandled_exception () at main.cpp:4
#2  0x0000555555555230 in main () at main.cpp:8
```

and from `#1` we can see that `throw` was called from `unhandled_exception` function at `main.cpp:4`.

If you know how to setup *Qt Creator* IDE to stop at `throw` while debugging leave me a commnet bellow.

And that is pretty much all for today, happy debugging!

[gdb]: https://www.gnu.org/software/gdb/
