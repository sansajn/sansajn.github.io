---
layout: post
title: "Task based concurrency in C++"
date: 2021-02-24 8:00:00 +0100
tags: [std, c++, c++11, concurrency]
comments: true
author: Adam Hlavatovic
---

To task based concurrency model C++ standard library (from C++11) offers these

```c++
packaged_task<F>
promise<T>
future<T>
shared_future<T>
x=async(policy, f, args)
x=async(f, args)
```

abstractions.

Classes `promise<T>` and `future<T>` implements shared state so it is possible to get result (including exceptions) from one thread to another in a safe way. Than we have `packaged_task<F>` there, which is basically wrapper around `promise<T>`/`future<T>` pair and allows to run functor without to care much about details. The third abstarction is `async()` function which implements thread luncher under the hood and is by far the easiest to use and it should be always the fist design choice.

The first and most easiest way to start with task based concurrency is with `async()` abstraction. Lets look how `async()` can be used

```c++
#include <future>
#include <iostream>
using std::cout;
using std::async, std::future;

int meaning_of_life()  // (1)
{
	return 42;
}

int main(int argc, char * argv[])
{
	future<int> answer = async(meaning_of_life);  // (2)
	cout << "The answer is " << answer.get() << "\n";  // (3)

	cout << "done!\n";
	return 0;
}
```

As we can see, we only need task in our case defined as an ordinary function `meaning_of_life()` (1) and executed by `async()` call which returns `future<int>` instance (2). The result is later acquired by `future<T>::get()` member function call (3).

The second abstraction is `packaged_tast<F>` which can be used this way

```c++
#include <future>
#include <thread>
#include <utility>
#include <iostream>
using std::packaged_task, std::future;
using std::thread;
using std::move;
using std::cout;

int main(int argc, char * argv[])
{
	packaged_task<int()> meaning_of_life{[](){return 42;}};  // (1)

	future<int> answer = meaning_of_life.get_future();  // (2)

	thread th{move(meaning_of_life)};  // (3)

	cout << "The answer is " << answer.get() << "\n";

	th.join();  // wait for thread

	cout << "done!\n";
	return 0;
}
```

As we can see there using `packaged_task<F>` is little a bit trickier, because now we need run our `packaged_task<int>` instance (1) in a thread (3), but before that we need to get `future<T>` by `packaged_task<F>::get_future()` member function call (2). The rest of the program is the same, the answer is acquired by calling `future<T>::get()` member function and ofcourse we need to `join()` the thread otherwise exception will be thrown.

The third and most trickier abstraction is `promise<T>`/`future<T>` pair, but on the other hand it also allows more control then `packaged_task<F>` or `async()` function. Lets look how `promise<T>`/`future<T>` pair can be used

```c++
#include <future>
#include <thread>
#include <utility>
#include <iostream>
using std::promise, std::future;
using std::thread;
using std::move;
using std::cout;

void meaning_of_life(promise<int> && answer)
{
	answer.set_value(42);  // (4)
}

int main(int argc, char * argv[])
{
	promise<int> answer_promise;  // (1)
	future<int> answer = answer_promise.get_future(); // (2)

	thread th{meaning_of_life, move(answer_promise)};  // (3)

	cout << "The answer is " << answer.get() << "\n";

	th.join();  // wait for thread

	cout << "done!\n";
	return 0;
}
```

This time instead of `packaged_task<F>` we create `promise<T>` (1) and ask for `future<T>` with `promise<T>::get_future()` member function call (2). Also this time we need to run our job represented by `meaning_of_life()` function in a thread (3) and we also need to move `promise<int>` instance so it can be later used to deliver the result to the calling thread by `promise<T>::set_value()` member function call (4). The rest of the probram is the same as before.
