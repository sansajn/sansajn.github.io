---
layout: post
title: "Thread safe async loop implementation with ASIO"
date: 2021-03-14 8:00:00 +0100
tags: [concurrency, asio, c++20, c++]
comments: true
author: Adam Hlavatovic
---

In the one of my previous post ([ASIO based thread implementation thoughts]({% post_url 2021-03-06-athread %})) I was thinking about ASIO based thread like implementation `athread`. I've spend some time experimenting with the design and realized that what we need there is more like asynchronous loop allows thread safe communication via posting function objects than thread like structure implementation. 

In my current design `athread` becomes `basic_async_loop<Executor>` template with an `Executor` template parameter and `asio_executor` as its implementation so resulting `async_loop` is defined as

```c++
using async_loop = basic_async_loop<asio_executor>;
```

Executor implementation si strightforward and need implement `stop()`, `run()`, `post()` and `post_with_result()` functions. In case `asio_executor` implementation looks this way

```c++
class asio_executor
{
public:
	asio_executor() : _idle{_io} {}
	void stop() {_io.stop();}
	void run() {_io.run();}

	template <typename F>
	void post(F && f) {
		asio::post(_io, f);
	}

	template<typename F>
	auto post_with_result(F && f) -> future<decltype(f())> {
		using result_type = decltype(f());
		auto p = promise<result_type>{};
		future<result_type> result = p.get_future();

		asio::post(_io, [result = move(p), func = move(f)]() mutable {
			result.set_value(func());
		});

		return result;
	}

private:
	asio::io_context _io;
	asio::io_context::work _idle;
};
```

where there is nothing new there, `post()` ans `post_with_resupt()` are implemented the same way as before.

Used `basic_async_loop<>` design allows replace whole ASIO library with for example concurrent queue implementation.

Implementation `basic_async_loop<>` looks this way

```c++
template <typename Executor>
class basic_async_loop
{
public:
	template <typename F>
	void post(F && f) {
		_io.post(move(f));
	}

	void async_run() {
		_th = jthread{[this](stop_token st){
			stop_callback stop_cb{st, [this]{
				_io.stop();
			}};

			_io.run();
		}};
	}

	template<typename F>
	auto post_with_result(F && f) -> future<decltype(f())> {return _io.post_with_result(move(f));}

	void request_stop() {_th.request_stop();}
	[[nodiscard]] stop_token get_stop_token() const noexcept {return _th.get_stop_token();}

	void join() {
		if (_th.joinable())
			_th.join();
	}

private:
	jthread _th;
	Executor _io;
};
```

where we borrow `request_stop()` and `join()` functions from `std::jthread` interface. In addition `basic_async_loop<>` defines `async_run()` function to run loop in a new thread.

Last time we developed a little brain/worker sample where with a `async_loop` there is no any change there so it still looks this way

```c++
class brain
{
public:
	brain() {_loop.async_run();}
	async_loop & loop() {return _loop;}
	bool need_more() {return true;}
	void work_done() {cout << "*"; flush(cout);}

private:
	async_loop _loop;
};

class worker
{
public:
	worker(brain & b) : _b{&b} {}

	void operator()()
	{
		// work hard ..., then notify
		std::this_thread::sleep_for(100ms);
		_b->loop().post([b = this->_b]{b->work_done();});
	}

private:
	brain * _b;
};

class robo_worker
{
public:
	robo_worker(brain & b) : _b{&b} {}

	void operator()(stop_token st)
	{
		future<bool> more;
		do
		{
			// work hard ..., then notify
			std::this_thread::sleep_for(75ms);
			_b->loop().post([b = this->_b]{b->work_done();});

			// need more?
			more = _b->loop().post_with_result([b = this->_b]{return b->need_more();});
		}
		while (!st.stop_requested() && more.get());
	}

private:
	brain * _b;
};

int main(int argc, char * argv[])
{
	brain b;

	vector<future<void>> results;
	for (int i = 0; i < 3; ++i)
		results.push_back(async(worker{b}));

	auto r = async(worker{b});

	worker w1{b}, w2{b};
	w1();
	w2();

	auto u = async(robo_worker{b}, b.loop().get_stop_token());

	for (auto & r : results)
		r.get();

	std::this_thread::sleep_for(500ms);

	b.loop().request_stop();

	u.get();  // wait for robo_worker

	cout << "\ndone!\n";
	return 0;
}
```

> full blown sample code can be found as [`async_loop.cpp`](https://github.com/sansajn/test/blob/master/boost/asio/async_loop.cpp)

As I had more time to spend with ASIO implementation, I've found that `post()` based communication comes with a price. To store handler in a type agnostic way there is memory allocation each time `post()` is called, which can be pretty expensive in some cases.
