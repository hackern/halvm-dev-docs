### `DomainInfo`
This module is about "Various pieces of useful information about the current domain".


#### Types and Data Structure
`DomId` is the 32 bit width identifier in fact, also typed with `newtype`. There are also some reader (parser) and writer related with this data type.

`DomainHandle` is actually an array of bytes. It is also an instance of `Storeable`, so we can use pointer to access it. (PROBLEM: What does it do, exactly?)

`SID` is "security identifier" (PROBLEM: What does it do, exactly?)

`VCPU` is "VCPU identifier".


`DomainInfoFlag` Type:

	data DomainInfoFlag = DomainDying    | DomainHVM     | DomainShutdown
	                    | DomainPaused   | DomainBlocked | DomainRunning
	                    | DomainDebugged


`DomainInfo` Structure:

	data DomainInfo = DomainInfo {
	    diDomain          :: DomId
	  , diFlags           :: [DomainInfoFlag]
	  , diTotalPages      :: Word64
	  , diMaxPages        :: Word64
	  , diShrPages        :: Word64
	  , diPagedPages      :: Word64
	  , diSharedInfoFrame :: MFN
	  , diCPUTime         :: Word64
	  , diNumOnlineVCPUs  :: Word32
	  , diMaxVCPUId       :: Word32
	  , diSSIDRef         :: SID
	  , diDomainHandle    :: DomainHandle
	  , diCPUPool         :: Word32
	  }

`DomainInfoFlag` and `DomainInfo` are both instances of `Storeable`, which defines the serialization and deserialization functions as well.

To summarize, the `DomainInfo` has recorded two kind of resources: memory and computation.


#### Getter, Setter and Interfaces
For every domain, there are four flags:

	data DomainFlags = DomainPrivileged
	                 | DomainInitial
	                 | DomainModuleIsMultiboot
	                 | DomainModuleIsAFrame

There are also functions for:

* `domainModuleStart`: Provide the start location for any accompanying modules.
* `domainInfo`, `domainProcessorContext`, `setDomainProcessorContext`: information extraction

FFI interfaces:

1. `getMagicStringPtr`
2. `getDomainFlags`
3. `getDomainModuleStart`
4. `domainModuleLength`

There are defined in `domain_info.c`, which is actually function interface to a external structure:

	extern start_info_t *system_start_info;

Which is defined in `<xen/xen.h>`