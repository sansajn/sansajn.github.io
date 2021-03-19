---
layout: post
title: "Interruptible threads (jthread) in C++20"
date: 2021-02-28 11:00:00 +0100
tags: [series, c++20, c++, concurrency, jthread, stop_token, condition_variable_any, stop_callback]
comments: true
author: Adam Hlavatovic
---

We have a new toy in *C++20* standard called `jthread`. `jthread` is a thread implementation which doesn't need to be joined (by calling `join()`) before its destruction and it is also interruptible (via `stop_token`).

### Content

- [Interruptible `jthread` usage](#interruptible-jthread-usage)
- [Interruptible `condition_variable_any` usage](#interruptible-condition_variable_any-usage)
- [Interruption with `stop_callback`](#interruption-with-stop_callback)


## Interruptible `jthread` usage

Let's see how `jthread` and `stop_token` can be used together

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

> full blown sample available as [`jthread_hello.cpp`](https://github.com/sansajn/test/blob/master/c%2B%2B/thread/jthread_hello.cpp)


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

> full blown sample available as [`jthread_with_condition_variable.cpp`](https://github.com/sansajn/test/blob/master/c%2B%2B/thread/jthread_with_condition_variable.cpp)


## Interruption with `stop_callback`

Let's say we want to interrupt something without `stop_token` support like `asio::io_context`. We want to run blocking `asio::io_context::run()` within a thread this way

```c++
asio::io_context io;
jthread t{[&io](stop_token stop){
	io.run();  // #1
}};

t.request_stop();  // #2
```

The problem there is that once `io.run()` is executed (1) we can not interrupt it with `stop_token` (2) because the call is blocking so `stop_token` state is never checked. That is exactly where `stop_callback` can be used, this way

```c++
asio::io_context io;
jthread t{[&io](stop_token stop){
	stop_callback stopper{stop, [&io]{
		io.stop();  // #3
	}};

	io.run();  // #1
}};

t.request_stop();  // #2
```

where calling `request_stop()` (2) calls `stopper` handler which execute `io.stop()` (3) so `io.run()` returns and thread can be interrupted.
