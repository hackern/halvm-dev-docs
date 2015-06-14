### Misc
#### What is `.hsc` file?
In `.hsc` file, there will be speciall CPP (C Pre-Processor) macro code. It is always used to better utilize the FFI capacity of HaLVM.

The added power includes:

	#include
	#ifdef
	#endif
	#define
	
For more details, [http://projects.haskell.org/cpphs/](http://projects.haskell.org/cpphs/)

#### What is `.hs-boot` file
This is about to compile mutually recursive modules in GHC.

Every cycle in the module import graph must be broken by a hs-boot file. Suppose that modules A.hs and B.hs are Haskell source files, thus:

	module A where
	import B( TB(..) )
	
	newtype TA = MkTA Int
	f :: TB -> TA
	f (MkTB x) = MkTA x

	module B where
	import {-# SOURCE #-} A( TA(..) )

	data TB = MkTB !Int

	g :: TA -> TB
	g (MkTA x) = MkTB x

Here A imports B, but B imports A with a {-# SOURCE #-} pragma, which breaks the circular dependency. Every loop in the module import graph must be broken by a {-# SOURCE #-} import; or, equivalently, the module import graph must be acyclic if {-# SOURCE #-} imports are ignored.

For every module A.hs that is `{-# SOURCE #-}` - imported in this way there must exist a source file A.hs-boot. This file contains an abbreviated version of A.hs, thus:

	module A where
	newtype TA = MkTA Int


For more details, refer to [https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/separate-compilation.html#mutual-recursion](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/separate-compilation.html#mutual-recursion)

