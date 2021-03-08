---
layout: post
title: "Interruptible threads (jthread) in C++20"
date: 2021-02-28 11:00:00 +0100
tags: [series, c++20, c++, concurrency]
comments: true
author: Adam Hlavatovic
---

There is new shiny `jthread` thread implementation there in C++20 allows thread interruption via `stop_token` instance. But that is not all, finally the `jthread` instance doesn't need to be joined (by calling `join()`) before its destruction as in a case of `thread`.

Lets see how `jthread` can be used

```c++
#include <thread>
#include <chrono>
#include <iostream>

using std::jthread, std::stop_token;
using std::cout, std::endl;
using namespace std::chrono_literals;

int main(int argc, char * argv[])
{
	jthread th{[]{cout << "hello from jthread!" << endl;}};

	// jthread doesn't need to be joined before exit

	jthread th_interrupt{[](stop_token stok){  // #1 interruptible thread
		for (int i = 0; i < 10; ++i)
		{
			std::this_thread::sleep_for(200ms);

			if (stok.stop_requested())  // #2
			{
				cout << "thread quit requested, quitting!" << endl;
				return;
			}

			cout << "counter: " << i << endl;
		}
		cout << "thread done" << endl;
	}};

	std::this_thread::sleep_for(1s);

	th_interrupt.request_stop();  // #3 signalize thread quit

	cout << "done!\n";
	return 0;
}
```

In case we want create interruptible thread `th_interrupt` the threads functor need to have `stop_token` instance as argument (1). We can later call `jthread::request_stop()` to signalize thread quit from outside of the thread (3). In thread functor we can use `stop_token::stop_requested()` to check interruption was signalized (2) and peacefully quit the thread.
