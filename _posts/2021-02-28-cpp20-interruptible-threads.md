---
layout: post
title: "Interruptible threads (jthread) in C++20"
date: 2021-02-28 11:00:00 +0100
tags: [series, c++20, c++, concurrency, jthread, stop_token, condition_variable_any]
comments: true
author: Adam Hlavatovic
---

[comment]: we have a new toy in c++20 ...

There is new shiny `jthread` thread implementation there in C++20 allows thread interruption via `stop_token` instance. But that is not all, finally the `jthread` instance doesn't need to be joined (by calling `join()`) before its destruction as in a case of `thread`.

### Content

- [Interruptible `jthread` usage](#interruptible-jthread-usage)
- [Interruptible `condition_variable_any` usage](#interruptible-condition_variable_any-usage)


## Interruptible `jthread` usage

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


## Interruptible `condition_variable_any` usage

We can use `stop_token` also with `condition_variable_any` instance to interupt waiting on condition variable this way

```c++
#include <thread>
#include <condition_variable>
#include <queue>
#include <iostream>
using std::jthread, std::stop_token, std::condition_variable_any, std::mutex, std::unique_lock;
using std::queue, std::size;
using std::cout, std::endl;
using namespace std::chrono_literals;

queue<int> jobs;
mutex locker;
condition_variable_any cv;

int main(int argc, char * argv[]) {
	jthread worker{[](stop_token stop){  // #1
		while (true) {
			int job = 0;

			{
				unique_lock lock{locker};
				if (!cv.wait(lock, stop, []{return !jobs.empty();}))  // #2
					break;

				job = jobs.front();
				jobs.pop();
			}

			// do something with job there ...
		}
	}};

	std::this_thread::sleep_for(100ms);  // wait for worker thread

	// before we create any job for waiting worker, let's ask worker to stop
	worker.request_stop();  // #3

	cout << "exiting without unexecuting any job!\n";

	return 0;
}
```

where we first create worker thread (1). Worker job is to execute jobs from synchronized queue `jobs`, but at the beginning the queue is empty so worker is waiting for condition variable notification (2). Then before we push any job to the queue we ask worker to stop via `request_stop()` member funciton call (3).


[comment]: let's say you have queue without stop_token support, ...
