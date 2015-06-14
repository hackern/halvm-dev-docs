Rebuild
---

A key point in develop HaLVM is to learn how to rebuild **efficiently** after some hackings.

Currently, the best advice from developers are mostly making a fresh build from source. But it would definitely cost a lot of time and energy. I am proposing to:

1. Seperate the rebuilding of HaLVMCore from HaLVM-GHC.
2. Reserve more stuff in `make clean` so we don't delete things that won't change much.
3. Make sure that after a clean build, we `make` and see `every is already done` rather than a lot of recompilation and failures.

