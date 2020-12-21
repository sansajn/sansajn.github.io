---
layout: post
title: "Parsing command line arguments - the easy way!"
date: 2020-12-17 16:00:00 +0100
categories: boost
author: Adam Hlavatovic
---

I was recently worked on a utility and wanted to add support for parsing commandline arguments, but didn't want to reinvent the wheel and that I've found `boost::program_options`.


## Requirements

Let's say we are working on video encoder application (`venc`) and we want `venc` to be configurable via command line arguments this way

```text
venc [--encoder h264|vp8, =h264][--verbose][--log PATH][--output,o PATH][--help] [--input]INPUT
```

where optional `--encoder h264|vp8, =h264` option set used video encoder to *h264* or *vp8* with *h264* as default. Optional `--verbose` option produce extended logging output, optional `--log PATH` save logging output to specified file `PATH`. Mandatory `--output PATH` option (or `-o`) save encoded input video `PATH` and positional `INPUT` (with optional `--input`) defines encoded video file name.

We also want from the library to generate nicely formated help message in case `--help` argument is used, this way


```text
./venc --help
Usage: venc [OPTION]... -o OUTPUT [--input]INPUT

Options:

  --help                 produce help messages
  --encoder arg (=h264)  used video encoder, available encoders are h264, h265,
                         vp8 or vp9
  --verbose              verbose output
  --log arg              create log file
  -o [ --output ] arg    save converted video as arg
```


## The Code

First include `boost/program_options.hpp` header file and define `po` namespace and some `using` so the code will not become messy, this way

```c++
#include <string>
#include <iostream>
#include <boost/program_options.hpp>

using std::cout, std::endl, std::string, std::exception;
namespace po = boost::program_options;
```

then inside `main` function create instance of `options_description` to describe our command line options

```c++
string opt_encoder;

po::options_description opts;
opts.add_options()
   ("help", "produce help messages")
   ("encoder", po::value<string>(&opt_encoder)->default_value("h264"),  // option with default value
      "used video encoder, available encoders are h264, h265, vp8 or vp9")
   ("verbose", po::bool_switch()->default_value(false), "verbose output")  // switch option
   ("log", po::value<string>(), "create log file")  // string value option
   ("output,o", po::value<string>()->required(), "save converted video as arg");  // required option

// ...
```

We want `--input` option to be positional one so instead

```bash
venc -o encoded.mkv --input foo.mp4
```

we can simply write

```bash
venc -o encoded.mkv foo.mp4
```

to run `venc`. We also want `--input` option to be mandatory and not shown to the user in a list of options in case of `--help` argument used. We need one set of options for parsing (`cmd_opts`) and one set of options for generating help message (`visible_opts`).

```c++
// options not shown to the user
po::options_description hidden_opts;
hidden_opts.add_options()
   ("input", po::value<string>()->required(), "input video file");

po::positional_options_description pos_opts;  // positional options
   pos_opts.add("input", 1);

po::options_description visible_opts{"Options"};  // to generate help message
   visible_opts.add(opts);

po::options_description cmd_opts;  // parsed options
   cmd_opts.add(opts).add(hidden_opts);
```

The next step is to parse command line arguments array defined by `argc` and `argv` variables from `main` function

```c++
// parse arguments
po::variables_map args;
po::store(po::command_line_parser{argc, argv}
   .options(cmd_opts)
   .positional(pos_opts)
   .run(), args);
```

all command line arguments are now available in `std::map` like variable `args` so options can be accessed with `[]` operator (like `args["output"]` or `args["log"]`, ...).

Generate notifications in case any mandatory argument is missing or `--help` argument used with

```c++
try {
   po::notify(args);
}
catch (exception & e)
{
   if (!args.count("help"))
      cout << "error: " << e.what() << "\n\n";

   cout << "Usage: venc [OPTION]... -o OUTPUT [--input]INPUT\n"
      << "\n"
      << visible_opts << "\n";

   return 1;
}
```

code.

The rest of the sample just print argument values this way

```c++
cout << "encoder=" << opt_encoder << "\n"
   << "log=" << (args.count("log") ? args["log"].as<string>() : "not used") << "\n"
   << "verbose=" << std::boolalpha << args["verbose"].as<bool>() << "\n"
   << "output=" << args["output"].as<string>() << "\n"
   << "input=" << args["input"].as<string>() << "\n";
```

and that is all.

Full sample source code available as [`venc_sample.cpp`](https://github.com/sansajn/test/blob/master/boost/program_options/venc_sample.cpp).


## Final Testing

Running `venc` with output and input arguments produce

```console
$ ./venc -o encoded.mkv foo.mp4
encoder=h264
log=not used
verbose=false
output=encoded.mkv
input=foo.mp4
done!
```

output. Running `venc` without mandatory output argument

```console
$ ./venc foo.mp4
error: the option '--output' is required but missing
```

cause missing output argument error. Running `venc` with verbose and encoder arguments produce

```console
$ ./venc -o encoded.mkv foo.mp4 --verbose --encoder vp8
encoder=vp8
log=not used
verbose=true
output=encoded.mkv
input=foo.mp4
done!
```

output.
