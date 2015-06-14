# Hypercalls

Before analyzing the code, what is **hypercall**?

From official document, the points are:

1. "Domains will use hypercalls to request privileged operations like updating pagetables" --> So, this is the way HaLVM manages its memory resources
2. "the hypercall is synchronous, but the return path from the hypervisor to the domain uses *event channels*"

There is also a [Hypercall Interface Specification](http://xenbits.xen.org/docs/unstable/hypercall/x86_64/index.html), it records all hypercalls clearly.

## Domain Control
This part is in the file `Hypervisor/Hypercalls/DomainControl`, in which some **domain control operations** are defined.

`domainControlOp` first allocate some buffer for maintain a `xen_domctl_t` structure. Then, some information, like what operation to do, what is the target of operation, is written on this structure. Finally, `setter` and `getter` are used to set the `argp` (argument pointer?) and to ????.

### Build*
There are also a bunch of `build*` functions, which, as I supposed, are used to turn haskell ideas into binary data structure which is used in Hypercall, such as:

1. `CreatDomainCall`
2. `ReadCPUAffinity`
3. `CPUAffinityReq`
4. `IRQPermissionReq`
5. `IOMemPermReq`
6. `IOPortPermReq`
7. etc.


## Physical Device
The structure is just like "Domain Control", just some transformation from Haskell to C interfaces. But the `setter` and `getter` are still mysterious.

Actually, in `Core`, only `Hypervisor/Platform.hs` used it:

	physicalDeviceOp PhysDevOpEOI (\ p -> poke p irq) (\ _ _ -> return ())

In this sentence, `EOI` command is used, with a setter write `irq` to the pointer, as well as a getter doing nothing as I supposed.

## System Control

Some other utilites:

1. `systemControlOp`
2. `buildReadConsoleCommand`
3. `buildGetDomInfoListReq`
4. `readGetDomInfoListResp`

The related system control operations are:

1. ReadConsole
2. PhysicalInfo
3. GetSchedulerId
4. GetDomInfoList

And some builders as well.

## Others
`updateVAMappingOtherDomain` and `populatePhysmap` are other two functions exported.

### TLB Effect
From `Hypervisor/OtherDomain.hs`, we will notice:

	data TLBEffect = NoFlush
	               | InvPage TLBTarget
	               | FlushTLB TLBTarget
	  deriving (Eq, Show)

	data TLBTarget = AllTLBs
	               | MultipleTLBs (VPtr CPUMap)-- ^Should be at least 8-byte aligned
	               | LocalTLB
	  deriving (Eq, Show)


These flags tell Xen how to handle the TLB when it does a virtual address mapping update.

As we all know, `TLB` (translation lookaside buffer) is a memory cache that stores recent translations of virtual memory to physical addresses for faster retrieval.

The `InvPage` might be meaning `invalid page"?

### `updateVAMappingOtherDomain` and `populatePhysmap`
Whatever, `updateVAMappingOtherDomain` should be used to "update the VA translation in a certain domain, while directing how to deal with the TLB.

`populatePhysmap` will set up the "reservation space", and do a `do_memory_op` with `XENMEM_populate_physmap` flag. Then, this reservation space is freed.

