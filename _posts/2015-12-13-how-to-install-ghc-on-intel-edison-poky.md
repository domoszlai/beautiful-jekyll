---
layout: post
title: How to install GHC on Intel Edison Poky
tags: [ghc, intel edison]
show-avatar: true
---

I acquired an Intel Edison recently as part of my plans of moving from Arduino to a platform with more computation power. Edison has enough power to enable writing the software of my future robots in a dependently typed language, and it also has an [MCU](https://en.wikipedia.org/wiki/Microcontroller) for the real time parts. Its power consumption is also low enough even for a small robot.

[GHC](https://www.haskell.org/ghc/) is not quite dependently typed (yet), but its type system is powerful enough to emulate it to an [acceptable extent](https://www.fpcomplete.com/user/konn/prove-your-haskell-for-great-safety/dependent-types-in-haskell).  

Anyway, first step first. I tried creating binaries on a Debian, but they crashed on the Edison, leaving me no choice, but compiling on the device itself. 

Installing a GHC on the Edison is more than straightforward, except that there is not much space on the `/home` partition, so downloading and uncompressing must be done in one step.

```sh
wget http://downloads.haskell.org/~ghc/7.10.2/ghc-7.10.2-i386-unknown-linux-deb7.tar.bz2 -O - | tar -xj
cd ghc-7.10.2
./configure 
make install
```

Enjoy!


