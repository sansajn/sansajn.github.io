---
layout: post
title:  "Loading text file to std::string?"
date:   2020-12-01 08:10:00 +0100
categories: boost filesystem
author: Adam Hlavatovic
---
In case we want to load text file to `std::string`, I've found that *Booost.Filesystem* has a support for it in [`string_file.hpp`](https://www.boost.org/doc/libs/1_74_0/boost/filesystem/string_file.hpp) header file. There are two functions defined there `load_string_file()` and `save_string_file()`. The first one loads file content to `std::string` and the second one saves `std::string` to text file.

Function `load_string_file()` can be used this way

```c++
#include <string>
#include <iostream>
#include <boost/filesystem/string_file.hpp>

using std::cout, std::string;
using boost::filesystem::load_string_file, boost::filesystem::path;

int main(int argc, char * argv[])
{
	string content;

	path p{argv[0]};
	p += ".cpp";

	load_string_file(p, content);

	cout << "file " << p << " is " << size(content) << " bytes long\n";

	cout << "done!\n";

	return 0;
}
```

where the program loads its source file as a `std::string` and print its length in bytes. For further details see [load_file.cpp](https://github.com/sansajn/test/blob/master/boost/filesystem/load_file.cpp) source and [SConstruct](https://github.com/sansajn/test/blob/master/boost/filesystem/SConstruct) build script sample.

