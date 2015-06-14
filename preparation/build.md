Build
---

> This file mainly explains how the current `Makefile` works. It is too long and and not very well written so it would be a little difficult to read.

The top-level `Makefile` mainly does:

1. Download the neccessary packages to build halvm-ghc
	* GHC 7.8.4 binary version
	* Cabal
2. Setup `halvm-ghc` build tools
	* Happy
	* Alex
	* HsColour
	* Haddock
3. Prepration of build the halvm-ghc
4. Bare-metal supportings and other tools
5. HaLVM building

## Download & Setup
Two things will be downloaded first, the binary ghc and cabal. We will setup up a cabal sandbox and fetch the packages of alex, happy etc. There are two kinds of build: In-sandbox building and static building. We will copy-install the static version for usage.

`PLATXXX` will all be prepared in this stage

## Preparation of GHC build
1. `sync-all` to get all GHC's libraries
2. Replace the default base library with halvm-specific base library
3. `boot` the GHC build tree
4. Link Xen system includings into `rts`
5. Link XenDevice library into `libraries`
6. Replace libc headers with minlibc

## Supportings
1. (Optional) Integer-GMP support
2. LibM -- Math library
3. LibIVC -- Inter-Domain Communication Support
4. `convert-profile` and `mkrenddir` tools
5. Bootloader code

## HaLVM building
With all GHC and happy, with all injected code in place, we can start to build the `stage1` compiler as well as RTS library code.

After that, some more runtime code will be compiled. Finally, all `halvm-XXX` is installed to the tools `bin` directory.


