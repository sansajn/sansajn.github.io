---
layout: post
title: "Working with ranges in C++20"
date: 2021-03-23 09:00:00 +0100
tags: [series, c++20, c++]
comments: true
author: Adam Hlavatovic
---

I was thinking on ranges for several years now, but working with them was not easy until they reached standard in *C++20* last year. There are already great introductory series about ranges like one from Rainer Grimm [C++20: The Ranges Library](https://www.modernescpp.com/index.php/c-20-the-ranges-library). I can definitely see the benefits of ranges, but I'm missing "real life" ranges samples or series. 

Couple of days ago, in [The Modern C++ Challenge](https://www.packtpub.com/product/the-modern-c-challenge/9781788993869) book, I've saw *find files* sample implemented with standard algorithms. The sample can recursively list files in current directory that match certain regular expression pattern. It was implemented this way

```c++
vector<fs::directory_entry> find_files(fs::path const & path, std::string_view expr) {
	vector<fs::directory_entry> result;
	regex rx{expr.data()};

	copy_if(
		fs::recursive_directory_iterator{path},
		fs::recursive_directory_iterator{},
		back_inserter(result),
		[&rx](fs::directory_entry const & entry){
			return fs::is_regular_file(entry.path()) && regex_match(entry.path().filename().string(), rx);
		}
	);

	return result;
}
```

where `fs` is defined as `namespace fs = std::filesystem`. I was curious how dificult can it be to implement the same with ranges library in *C++20*. The `find_files()` function implementation looks this awy

```c++
auto find_files(fs::path const & path, string_view pattern) {
	regex e{pattern.data(), size(pattern)};
	return subrange{fs::recursive_directory_iterator{path}, fs::recursive_directory_iterator{}}
		|filter([e](fs::directory_entry const & x){
	 		return fs::is_regular_file(x.path()) 
				&& regex_match(x.path().filename().string(), e);
	 	});
}
```

it is shorter and somehow easier to read (at least after some time spend wirh ranges). I've learned several thinks there 

- `subrange` can be used to create range from pair of iterators
- we can compose ranges with pipe operator `|`
- we need to be carefull capture by value and not by reference when lamdas are used together with ranges (yes, I did the mistake with `[&e]` there)
- composed range type is better to handle with `auto`

Full sample looks this way

```c++
#include <string_view>
#include <filesystem>
#include <regex>
#include <iostream>
#include <ranges>
using std::string_view, std::cout;
using std::regex, std::regex_match;
using std::ranges::subrange, std::ranges::views::filter;
namespace fs = std::filesystem;

auto find_files(fs::path const & path, string_view pattern) {
	regex e{pattern.data(), size(pattern)};
	return subrange{fs::recursive_directory_iterator{path}, fs::recursive_directory_iterator{}}
		|filter([e](fs::directory_entry const & x){
	 		return fs::is_regular_file(x.path()) 
				&& regex_match(x.path().filename().string(), e);
	 	});
}

int main(int argc, char * argv[]) {
	auto pattern = argv[1];
	auto r = find_files(fs::current_path(), pattern);
	for (auto const & x : r)
		cout << x.path() << "\n";
	return 0;
}
```

> full blown find files sample available as [`find_files.cpp`](https://github.com/sansajn/test/blob/master/c%2B%2B/ranges/find_files.cpp)

Further reading :

- list of [ranges algorithms](https://en.cppreference.com/w/cpp/algorithm/ranges) and
- ranges library [reference](https://en.cppreference.com/w/cpp/ranges).
