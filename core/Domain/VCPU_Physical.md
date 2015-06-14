> This section is more or less hardware-specific, which might not need a lot attentions.

## VCPU

> From web: A virtual CPU (vCPU) also known as a virtual processor, is a physical central processing unit (CPU) that is assigned to a virtual machine (VM).
By default, virtual machines are allocated one vCPU each. If the physical host has multiple CPU cores at its disposal, however, then a CPU scheduler assigns execution contexts and the vCPU essentially becomes a series of time slots on logical processors.

`FPUContext` is the state of the FPU (Float Point Unit), generated from something like FXSAVE.

Processor Flag:

	data ProcessorFlag = ProcessorI387
	                   | ProcessorInKernel
	                   | ProcessorFailsafeDisablesEvents
	                   | ProcessorSyscallDisablesEvents
	                   | ProcessorOnline

And the `ProcessorUserRegs`, which includes all the registers that user program might use.

And other definitions:

	data TrapInfo = TrapInfo {
	    tiVector    :: Word8
	  , tiFlags     :: Word8
	  , tiCS        :: Word16
	  , tiAddress   :: Word
	}

	data VMAssistFlag = VMAssist4GBSegments
	                  | VMAssist4GBSegmentsNotify
	                  | VMAssistWritablePageTables
	                  | VMAssistPAEExtendedCR3

And the whole context:

	data ProcessorContext = ProcessorContext {
	    rcFPU                :: FPUContext
	  , rcProcFlags          :: [ProcessorFlag]
	  , rcUserRegs           :: ProcessorUserRegs
	  , rcTrapInfo           :: [TrapInfo]
	  , rcLDTBase            :: Word
	  , rcLDTEntries         :: Word
	  , rcGDTFrames          :: [Word]
	  , rcGDTEntries         :: Word
	  , rcKernelSS           :: Word
	  , rcKernelSP           :: Word
	  , rcControlRegs        :: [Word]
	  , rcDebugRegs          :: [Word8]
	#ifdef x86_64_TARGET_ARCH
	  , rcEventCallbackIP    :: Word
	  , rcFailsafeCallbackIP :: Word
	#else
	  , rcEventCallbackCS    :: Word
	  , rcEventCallbackIP    :: Word
	  , rcFailsafeCallbackCS :: Word
	  , rcFailsafeCallbackIP :: Word
	#endif
	  , rcVMAssist           :: [VMAssistFlag]
	#ifdef x86_64_TARGET_ARCH
	  , rcFSBase             :: Word64
	  , rcGSBaseKernel       :: Word64
	  , rcGSBaseUser         :: Word64
	  , rcSyscallCallbackIP  :: Word
	#endif
	  }

One thing to notice is that, using CPP extension, we can make Haskell code architecture specific.

## `CPUMap`
This is a module which lifts a low-level CPUMap to Haskell.

From this definition `data CPUMap = CPUMap Int Integer`, we can see that a mapping is a pair of integer. But what does this map means?

The first `Int` is the number of avaiable cpus, and the second one is a bit-map of allocation condition.

All utilities defined:

* Create an empty CPU map with the maximum CPU of (n - 1).
* Remove all CPUs from the map
* Turn on all CPUs in the map
* Add the given CPU number to the map
* Remove the given CPU number from the map
* Return the maximum CPU number the map can contain
* Union together two maps
* Intersect two CPU maps.
* Synthesize a CPUMap serialization routine for use within the given function.
* Read a serialized CPUMap into our internal format.
* The size of a serialized CPU map in memory, not counting the internal data structures.


The amazing thing here is that it is not "timeslice" that is mapped to "processes", but "cpus" mapped to "domains". Actually, in `Control.hs`, we have 

	getDomainCPUAffinity :: DomId -> VCPU -> IO CPUMap
	setDomainCPUAffinity :: DomId -> VCPU -> CPUMap -> IO ()
