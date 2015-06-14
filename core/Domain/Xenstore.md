## Domain
"Domain" is a concept of Xen. The domain0 is the privileged, firstly booted OS, which is just like a common Linux System. DomainU is the "User Domain", which can run any system with Xen support.

### `XenStore`
`Xenstore` is an information storage space shared between domains maintained by `Xenstored`. It is meant for configuration and status information rather than for large data transfers.

It can be checked on Domain0 with `xenstore-ls` command.

A related concept is `XenBus`, which provides a bus abstraction for paravirtualized drivers to communicate between domains. In practice, the bus is used for configuration negotiation, leaving most data transfer to be done via an interdomain channel composed of a shared page and an event channel.

#### Permissions -- `XSPerm`

Any guest can read any part of the store, but only if it has permission to do so. Domain 0 may read or write anywhere in the store, regardless of permissions, and permissions are set up by the tools in domain 0, or by Xenstored when it first starts up.

#### Transaction -- `TransId`
Transactions provide developers with a method for ensuring that multiple operations on the Xenstore are seen as a single atomic operation. Any time multiple operations must be performed before any changes are seen by watchers, a transaction must be used to encapsulate the changes.

#### MFN and Xen's Address Spaces
* **MFN** (Machine Frame Number): Real Host Machine Address, for CPU
* **GPFN** (guest pseudo-physical frame number): Guests run in an illusory contiguous physical address space, which is probably not contiguous in the machine address space.
* **GMFN** (guest machine frame number): Equivalent to GPFN for an auto-translated guest, and equivalent to MFN for normal paravirtualised guests. It represents what the guest thinks are MFNs.
* **PFN** (physical frame number): A catch-all for any kind of frame number. "Physical" here can mean guest-physical, machine-physical or guest-machine-physical.

#### Request
standardRequest

    standardRequest :: MVar XenbusState ->
                       TransId -> XenbusRequest ->
                       ResponseType -> (ByteString -> a) ->
                       IO a
    standardRequest stateMV tid req goodresp converter =
      writeRequest stateMV tid req $ \ body -> case body of -- ...

XenbusState: 复习：组成原理里的总线

    data XenbusState = XBS {
        xbPort          :: Port
      , xbRing          :: XSRing
      , nextRequestId   :: ReqId
      , decodeStream    :: ByteString
      , pendingWrites   :: ByteString
      , waitingRequests :: Map ReqId (MVar ResponseBody)
      , waitingWatches  :: [(String, String, String -> String -> IO ())]
	}
    
XenbusRequest

    data XenbusRequest =
        XSRead     String
      | XSWrite    String String
      | XSMkDir    String
      | XSRm       String
      | XSDir      String
      | -- etc.

ResponseType

    data ResponseType = RTRead     | RTWrite  | RTMkDir   | RTRm       | RTDir
                      | RTSetPerms | RTWatch  | RTUnwatch | RTRestrict | RTTransSt
                      | RTTransEnd | RTIntro  | RTRelease | RTGetPath  | RTError
                      | RTIsDomInt | RTResume | RTSetTarg | RTGetPerms | RTEvent

Given a state, a transaction, a request, a responset type adn a converter, we will have a standard request.

I assume that "directory" is a way to identify a transaction, but there is no Xen's documentation on that.


    xsDirectory :: XenStore -> String -> IO [String]
    xsDirectory xs = xstDirectory xs emptyTransaction

    xstDirectory :: XenStore -> TransId -> String -> IO [String]
    xstDirectory (XenStore xbs) tid str =
	  standardRequest xbs tid (XSDir str) RTDir parseStrings

However, all such operations, including: `xsStartTransaction`, `xstCommit`, `xstIntroduce`, `xstRelease`, `xstGetDomainPathuse`, `xstSetPermissions` the same kind of logic -- Wrap a standard request.

#### Operations on Request and Response
These operations are mostly internal to the `XenStore` module, including some read, write, parse, build, render etc.
