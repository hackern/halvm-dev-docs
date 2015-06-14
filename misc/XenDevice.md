# XenDevice

XenDevice is a cabal package. The introduction:

> This package provides access to the Xen paravirtualized devices; virtual disks, virtual network cards, the virtual console, and the Xenbus/Xenstore. In the future, it will also work with virtualized TPMs. For those interested in creating their own devices, the XenDevice library exports a module for implementing drivers over the standard Xen ring buffer protocol, as well.

So, what are the devices:

1. RingBuffer
2. Disk
3. NIC (Network Interface Controller)

## Disk
Disk device provides a low-lovel interface to disk, so you can implement a file system on top of it.

`advanceReqId` seems weird, since it seems to be equivalent to `(+1)`

`openDisk` used `findDevice` to get a pair of "frontend" and "backend".   However, I just saw that they hold different kinds of meta-info.

A very awesome line is:

	  _ <- forkIO $ forever $ processResponses ring table

What does `xsRead` do? It should be a way to read from `XenStore`.

And there are so much details about request/response model, about extensive use of `MVar`, about segment and sector etc., so I will not cover these here since I am an not expert on writing disk driver.

## NIC
As Adam wrote:

> Network devices are initialized using the same two-phase system as disks. The initial boot process discovers a series of potential network devices. Later, a program (or more likely, a network stack) may attempt to connect to the network device. If successful, it can then use it to send and receive packets.

The `listNIC`s will return a list of found entities under `device/vif` directory.

The `openNIC` can return a `NIC` based on mac address.

The `sendPacket` can write a `ByteString` to a `NIC`.

And there is `setReceiveHandler` as well. Look at its code:

```haskell
setReceiveHandler nic handler = do
  _ <- takeMVar (nicRxHandler nic)
  putMVar (nicRxHandler nic) handler
```

This is just a substitution of handler (`ByteString -> IO()`)!

## PCI

