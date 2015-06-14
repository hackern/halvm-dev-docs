# IVC

IVC: Inter Virtual-machine Commmunication, or inter-domain communication for Xen. The highlight is that channels are **typed**.

In this module, there are the input and ouput ends of open unidirectional communication channels, or bidirectional channels. These channels are parameterized over the types of messages that can be sent over them, so that domains can exchange messages in a type-safe manner.

The types are: InChannel, OutChannel, InOutChannel, ReadableChan, WriteableChan.

So, why I/O channel are different from R/W channel?


And the composition of a channel is interesting:

```haskell
data InChannel a = InChannel {
    ichSetupData :: Maybe (DomId, [GrantRef], Port)
  , ichInChannel :: InChan
  , ichPeer      :: DomId
  }
```

The `inChan` is defined internally, as that:

```haskell
data InChan = InChan {
    icBuffer    :: Ptr Word8
  , icSize      :: Word
  , icModulus   :: Word32
  , icPort      :: Port
  , icStateMV   :: MVar InChanState
  }
```

, in which states includes:

```haskell
data InChanState = NeedSize [MVar ByteString]
                 | GotSize !Word32 ByteString [MVar ByteString]
```

## R/W v.s. I/O

```haskell
buildIOChan :: Float -> Word ->
               Bool -> Ptr Word8 -> Word -> Port ->
               IO (InChan, OutChan)
buildIOChan perc npages doClear ptr _ port = do
  let p1Size   = floor ((fromIntegral (npages * 4096)) * perc)
      p2Size   = (npages * 4096) - p1Size
      b1Size   = p1Size - bookkeepingOverhead
      b2Size   = p2Size - bookkeepingOverhead
  let (inPtr, inSize, outPtr, outSize) =
        if doClear -- The position of two segments are reversed
          then (ptr, b1Size, ptr `plusPtrW` p1Size, b2Size)
          else (ptr `plusPtrW` p1Size, b2Size, ptr, b1Size)
  ichn <- buildRawInChan  doClear inPtr  inSize  port
  ochn <- buildRawOutChan doClear outPtr outSize port
  setPortHandler port $ tryWriteData ochn >> tryReadData ichn
  return (ichn, ochn)
```

In this code, we partition some pages as buffers, then build a `ichn` and a `ochn`. The interesting  thing is the `setPortHandler`, which installs a function that will be invoked whenever the HALVM receives an event on the given channel (composite of I/O channels?)

```haskell
class WriteableChan c a | c -> a where
  put :: c -> a -> IO ()

instance Binary a => WriteableChan (OutChannel a) a where
  put c = putBinary (ochOutChannel c)

instance Binary b => WriteableChan (InOutChannel a b) b where
  put c = putBinary (bchOutChannel c)
```

Now, above are the definitions of R/W. `| c -> a` is a dependent type notation. It could means that, the channel `c`'s type will affect the type of data `a` to write. This could be more clear from the instance signature `(OutChannel a) a`, in which `a` is used twice.

```haskell
putBinary :: Binary a => OutChan -> a -> IO ()
putBinary oc x = runWriteRequest oc (encode x)
```
We can see that, for R/W channel, we can use a `put` on it.

## Buffer
A communications channel is composed of something of a pair of a pointer and a size, where:

	+-----------------------+ ptr + 0
	+         ...           |
	+         ...           |
	+     buffer space      |
	+         ...           |
	+         ...           |
	+-----------------------+ ptr + size
	+    bytes consumed     |
	+-----------------------+ ptr + size + 4
	+    bytes produced     |
	+-----------------------+ ptr + size + 8

The reading/writing are all depending on the basic operations defined here. Let's just analyze: 

```haskell
runReadRequest :: InChan -> IO ByteString
runReadRequest ich = do
  resMV <- newEmptyMVar
  istate <- takeMVar (icStateMV ich)
  case istate of
    NeedSize waiters ->
      putMVar (icStateMV ich) $! NeedSize (waiters ++ [resMV])
    GotSize n acc waiters ->
      putMVar (icStateMV ich) $! GotSize n acc (waiters ++ [resMV])
  tryReadData ich
  takeMVar resMV
```

First, a `MVar` is initialized, than, another `MVar` is taken out. There are two possibility, and based on where `NeedSize` or `GotSize`, the new `MVar` is attached in the state.

Then, we will `tryReadData`, which is a rather long function. Just see its mainbody:

```haskell
tryReadData :: InChan -> IO ()
tryReadData ich = modifyMVar_ (icStateMV ich) $ \ istate -> do
  prod             <- bytesProduced (icBuffer ich) (icSize ich)
  cons             <- bytesConsumed (icBuffer ich) (icSize ich)
  (istate', cons') <- doPossibleReads prod cons istate
  setBytesConsumed (icBuffer ich) (icSize ich) cons'
  when (cons /= cons') $ sendOnPort (icPort ich)
  return istate'
```

Basically, it `doPossibleReads` and set some flags. Then, when the consumption is different, we will `sendOnPort`, which is:

	sendOnPort (Port p) = evtchn_send (fromIntegral p) >>= standardUnitReturn

After that, the new state after `doPossibleReads` are returned.