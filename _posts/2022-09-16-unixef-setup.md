---
layout: post
title: "Unifex (task based concurrency) library setup"
date: 2022-09-16 8:00:00 +0100
tags: [unifex, concurrency]
comments: true
author: Adam Hlavatovic
---

Today I would like to describe setup for [unifex](https://github.com/facebookexperimental/libunifex) library. Unifex is library to supoprt *task based concurrency* and serves as a base for upcoming *C++ 23* executors library. The man behind unifex is Eric Niebler and I heard about the library from his Cppcon talks *Working with Asynchrony Generically: A Tour of C++ Executors* [part1](https://www.youtube.com/watch?v=xLboNIf7BTg) and [part2](https://www.youtube.com/watch?v=6a0zzUBUNW4&t=21s).

## Setup & install

The first step is to 

- install dependencies with

```bash
sudo apt install git ninja-build cmake gcc
```

command, then

- download the library with

```bash
git clone https://github.com/facebookexperimental/libunifex.git
```

command, then

- configure the library with

```bash
cmake -G Ninja -S libunifex -B build-libunifex -DCMAKE_CXX_FLAGS:STRING=-fcoroutines -DCMAKE_CXX_STANDARD:STRING=20
```

> works with GCC 11 (under *Ubuntu 22.04 LTS*)

command, then

- build the library with  

```bash
cmake --build build-libunifex
```

comamnd into `build-libunifex` directory. Then

- (optionally) test the library with

```bash
cd build-unifex
ctest
```

commands. Then

- install unifex with

```bash
cd build-unifex
sudo ninja install
```

commands into `/usr/local` (`lib`, `include/unifex`, ...) directory.

## Sample

Unifex *hello word* `execute.cpp` can looks this way

```c++
#include <iostream>
#include <unifex/single_thread_context.hpp>
#include <unifex/execute.hpp>
#include <unifex/scheduler_concepts.hpp>

using std::cout;
using unifex::single_thread_context, unifex::execute;

int main(int argc, char * argv[]) {
	single_thread_context ctx;

	for (int i = 0; i < 5; ++i) {
		execute(ctx.get_scheduler(), [i](){
			cout << "hello execute() " << i << '\n';
		});
	}

	return 0;
}
```

and *CMake* build script `CMakeLists.txt` can looks this way

```cmake
cmake_minimum_required(VERSION 3.16)
project(unifex-test CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_compile_options(-Wall -Wextra)

# we are using unifex version 0.1
find_package(unifex REQUIRED)

add_executable(execute execute.cpp)
target_link_libraries(execute PRIVATE unifex)
```

Save both files (`execute.cpp` and `CMakeLists.txt`) into `unifex` directory and run

```bash
cmake -S unifex -B build-unifex
cmake --build build-unifex
```

commands to build and

```bash
./build-unifex/execute
```

to run the sample.
