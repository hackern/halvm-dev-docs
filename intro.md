HaLVM -- An Introduction
---

The Haskell Lightweight Virtual Machine (or, informally, the HaLVM) is a port of the GHC runtime system for Haskell to barebones Xen. This means that Haskell programs written for the HaLVM run natively on Xen, without any intervening operating system, which allows them to boot quickly and use very little space.

Now it is mostly a combination of "Xen interface" and "Bare-metal GHC", with some nice extensions such as:


1. IVC mechanism
2. Halfs file-system
3. HaNS network-stack

Now, the GHC it supports is 7.8.3.

### Comparison with House
House is another try on building an OS with Haskell. Here is a little comparision between House and HaLVM

* HaLVM is in active development and supports GHC 7.8.3, while House is a little outdated. The Lighthouse project is done in 2010 and used GHC 6.8.2 version. And you can also get a 2009 version of House which has Gadgets support in GitHub.
* HaLVM is more "lightweight" and more hardware-compatible since it is built on Xen. House is much more heavier with a bunch of C code and bare-metal supporting code.
* HaLVM is another example (see MirageOS) of the concept "Library OS", which House is more like an complete traditional OS. For example, House should take care of paging but HaLVM doesn't have to.
* HaLVM is more for real-purpose usage while House is more academic. HaLVM doesn't even have a paper for it. (But some academic projects built itself on top of HaLVM)

