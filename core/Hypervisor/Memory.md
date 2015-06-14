# Memory

This file will include the memory management interfaces defined in `Hypervisor/Memory.hs`

This is said to be "A low-level module for dealing with unprivileged Xen memory operations, including allocating pages, granting access to pages to other domains, and mapping the grants of other domains."

There are four kinds of memory address:

1. `PFN`: Pseudo-Physical Frame Number: very little to do with the machine address or the virtual address, but are used in some Xen hypercalls.
2. `MFN`: Machine Frame Number, These frame numbers identify a phyical 4096-byte (4K) frame on the underlying hardware.
3. `VPtr`: Virtual Address, if you've mapped it, can be written to and read from as per normal.
4. `MPtr`: Machine Address, These cannot be written to or read from directly, as HaLVM's always run with paging enabled.

The main mapping is between **machine address** and **virtual address**. The paging is a two-level structure. The `MFN` is the frame identifier, which is one `Word` long (30 bits I thought). Plus the inner-page 4k or 12-bit wide addressing space, it could make a 42 bit wide space.

But in fact, `MPtr` could be `Word64` or `Word32` based on what hardware platform it is on. Interesting.

> Me: The `MFN` is a `Word`, and one page is 4 KB (12 bit wide), so does this means that we can totally address 12 + 30 = 42 bits wide machine memory space?

> Adam: Yes. It occurs to me that in very, very large machines this may be an issue, if MAXPHYSADDR is > 0x40000000000.

> Me: And what’s more, when we do `virtualToMachine`, the last step will `MPtr` the translated address. But it is possible that `MPtr` is just a `Word32`, so will there be any information lost?

> Adam: Take a look at the definition of MPtr: For PAE and 64-bit systems, an MPtr is defined as a Word64. So no worries on that front. (Actually, I don’t think Xen supports 32-bit non-PAE any more, so that #ifdef may be unnecessary at this point.)


## Page Operations
Every page in HaLVM is attached with **protection flags**, so all the operations must take this into consideration.

	data AllocProt = AllocProt
	  { protRead    :: Bool
	  , protWrite   :: Bool
	  , protExec    :: Bool
	  , protNoCache :: Bool
	  }

The data structure of a `page` is maintained by Xen I supposed, so we mainly see verbs here rather than nouns.

The operations includes setting permissions, create page or free page, do something with page and then free automatically (a little functional style here).

And there are also some *dangerous* operations, which could make some page as a page table, and pin it (`markAsPageTable`). On the other hand, also a `mapFrames` which could "map the given list of frames into a contiguous chunk of memory."

As a note, there is a quick guide on *levels of paging*:

-  Use level '1' for page tables
-  Use level '2' for page directories
-  Use level '3' for PAE base tables / directory pointer tables
-  Use level '4' for PML4


## Grant
A short intro:

> Each domain has its own grant table. This is a data structure that is shared with Xen; it allows the domain to tell Xen what kind of permissions other domains have on its pages. Entries in the grant table are identified by grant references. A grant reference is an integer, which indexes into the grant table. It acts as a capability which the grantee can use to perform operations on the granter’s memory.

To learn more about Xen's "Grant Table" memory sharing mechanism, go to [http://wiki.xen.org/wiki/Grant_Table](http://wiki.xen.org/wiki/Grant_Table)

In HaLVM, `GrantRef` is a wrapped `Word32` data type.

It is said that, grant access to a given domain to a given region of memory.

As far as `GrantHandle`, should be an array of granted references (Not certain yet actuall).

There are also `unmapGrant`, which Unmap the grant of another domain's page. This will make the shared memory inaccessible.

There are also some transfering functions, which could move one page or directory from one domain to another.

