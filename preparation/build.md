HaLVM Build Problems
===

## The basic process and some notices
I built HaLVM from source on GitHub.

Although the source is not large, but the `Makefile` will download **a bunch of stuff** from Internet. So please prepare a non-GFW networking environment before compiling.

The Galois Inc. says they tested on 64-bit, Xen 4.2, Fedora 19-based systems, but also:

> it works on my machine, sometimes, and I'm hoping it works on yours, sometimes.

I compiled it on my fedora VM and the only compilation problem I encountered should be:

* `llvm` (it seems not a problem, I found it when I recompiled again)
* `openssl-devel`
* `cabal` (fedora's `yum` don't have it in source, so install `haskell-platform` might be a choice)

And a final thing to remember: the `/tmp` on my fedora box is only 1.1G initially, which will be filled entirely at the very first building process (that might be a bug). So, you might needs to enlarge it before continuing.

