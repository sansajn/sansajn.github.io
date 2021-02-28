---
layout: post
title: "Reader-writer lock with std::shared_mutex in C++17"
date: 2021-02-28 14:00:00 +0100
tags: [series, c++17, c++]
comments: true
author: Adam Hlavatovic
---

*C++17* is shipped with `std::shared_mutex` which allows to implement reader-writer lock the easy way. There are also two lock helpers in `std`, `shared_lock` for reader threads and `unique_lock` for writer threads.

> **tip**: in *C++14* `shared_mutex` functionality is available as `shared_timed_mutex`

There is the sample

```c++
#include <shared_mutex>
#include <thread>
#include <string_view>
#include <vector>
#include <chrono>
#include <iostream>

using std::shared_mutex, std::shared_lock, std::unique_lock;
using std::thread;
using std::string_view;
using std::vector;
using std::cout, std::endl;
using namespace std::chrono_literals;

shared_mutex content_locker;  // #1

void reader(char initial)
{
	for (int i = 0; i < 10; ++i)
	{
		{
			shared_lock lock{content_locker};  // #2
			std::this_thread::sleep_for(2ms);
			cout << initial;
		}

		std::this_thread::sleep_for(10ms);
	}
}

void writer(vector<string_view> const & text)
{
	for (string_view const & w : text)
	{
		{
			unique_lock lock{content_locker};  // #3 whole world needs to be written without an interruption
			for (auto ch : w)
			{
				std::this_thread::sleep_for(2ms);
				cout << ch;
			}
			cout << " ";
		}

		std::this_thread::sleep_for(5ms);
	}
}

vector<string_view> text{
	"The", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog!"};

int main(int argc, char * argv[])
{
	thread w{writer, text};

	thread r1{reader, '.'},
	 	r2{reader, ','},
	 	r3{reader, ';'},
	 	r4{reader, ':'},
	 	r5{reader, '/'};

	w.join();
	r1.join();
	r2.join();
	r3.join();
	r4.join();
	r5.join();

	cout << "\ndone!\n";
	return 0;
}
```

where synchronization is acquired via `shared_mutex` instance `content_locker` variable (1), `shared_lock` guard instance for reader treads (2) and `unique_lock` guard instance for writer threads (3).

Generated output looks this way

```console
./reader_writer_lock
The :/.;,quick /:,.;brown :,.;/fox /:,;.jumps :/.;,over :;,./the :,.;/lazy :,;./dog! ,/;.::,.;/
done!
```

and in case you comment out (2) and (3) lines it looks this way

```console
./reader_writer_lock
T:./,;he q:./,;uick :/,.;brow:n /,.;fox :/;.,jum:/,.;ps o:/,.;ver :t/,.;he l/:,.;azy /:,.;dog!
done!
```

where we can clearly see writer thread is interrupted by reader threads in the middle of word writing.

