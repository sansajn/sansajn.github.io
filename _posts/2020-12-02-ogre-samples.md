---
layout: post
title:  "Building OGRE 3D samples."
date:   2020-12-02 10:22:00 +0100
categories: boost filesystem
---
OGRE samples are not part of standard `libogre-1.12` or `ogre-1.12-tools` packages (in *Ubuntu 20.04 LTS*) so we need to build samples by are own from source, but don't worry it is easy.

## 1. download *OGRE* source code 

Download *OGRE* source code with 

```bash
git clone https://github.com/OGRECave/ogre.git
```

command.

## 2. setup

- switch to the 1.12.4 *OGRE* branch with

```bash
cd ogre
git switch -c v1.12.4
```

command.

> you can list all available tags with `git tag` command

- and install dependencies with

```bash
sudo apt install libxaw7-dev libgles2-mesa-dev libxt-dev libxaw7-dev libsdl2-dev
```

## 3. building

Finally buid *OGRE* with

```bash
mkdir build
cd build
cmake ..
make -j6
```

commands.

## 4. running sample browser

After successfull build `SampleBrowser` can be located in `build/bin` directory so execute

```bash
cd bin
./SampleBrowser
```

commands to run sample browser.

After selecting one of the available renderer in a dialog window you should see

![SampleBrowser screenshot](/assets/image/ogre_sample_browser.jpg)

Adam Hlavatovic