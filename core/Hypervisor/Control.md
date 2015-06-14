# Control

Another file on domain:

> Various ways to control domains in execution. Most of these routines will require the domain to be privileged in order to execute on anythinge other than itself.


Most of the functions are just wrappers for `domainControlOp`.

```haskell
createDomain :: Maybe DomId -> SID -> DomainHandle -> [CreateFlag] -> IO DomId
destroyDomain :: DomId -> IO ()
pauseDomain :: DomId -> IO ()
unpauseDomain :: DomId -> IO ()
hypercallInit :: DomId -> MFN -> IO ()
setDomainHandle :: DomId -> DomainHandle -> IO ()
getDomainCPUAffinity :: DomId -> VCPU -> IO CPUMap
setDomainCPUAffinity :: DomId -> VCPU -> CPUMap -> IO ()
setDomainMaxVCPUs :: DomId -> Word32 -> IO ()
```