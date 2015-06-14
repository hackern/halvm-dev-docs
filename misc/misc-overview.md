# Misc

First, a lot of stuff under `misc`, such as `*.mk`, `*.mk.in` and `.lds` are just Makefiles and loader script. So why are they not put under these directory?

Except for the undecipherable scripts, there also `ghci_runtime.c` and `HsBase.h`.

From the `ghci_runtime.c` file, it is difficult to read its attempt, and there is no downstream files as well. But from GitHub commit history, we can see "Initial template haskell support".

`HsBase` is clear, it provides some basic operations.