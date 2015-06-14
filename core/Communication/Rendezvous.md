# Rendezvous

This module contains routines for automatically performing rendezvous between two domains.

What is **rendezvous**?

From [Wikipedia Rendezvous protocol](http://en.wikipedia.org/wiki/Rendezvous_protocol), it has several characteristics:

1. P2P
2. Handshaking Model

But the interesting thing is that, except for `peerConnection`, there is also a `clientServerConnection`, so does that mean that they are both under the category of "Rendezvous"?


## Path
If you read the source, you will find something called "path". What does that means?

Path is just a `String` essentially. From [this page](http://wiki.xen.org/wiki/XenStore_Reference), it said that "XenStore is a hierarchical namespace". And from [this page](http://xenbits.xen.org/docs/unstable/misc/xenstore-paths.html), we can see more about "path" concepts in Xen.

For example:

	~/domid = INTEGER []
	~/device/suspend/event-channel = ""|EVTCHN [w]
	~/data/* [w]

I guess that "path" can be constructed arbitrarily. It is just a identifier, or a protocal, or a way to send/receive some basic configuration-related information.

## PeerConnection
Let's illustrate the two sides' actions in the source code:

First, left side (abbr. as ls) sent up the "permission" path including its `domID`, than it `waitForKey` to appear. Now, rs will be notified about the appearing of `/LeftDomId`, so it made a connection based some stuff, and write the returned information needed by ls to the path. As long as something appears (grants, ports actually), it will confirm the connection response, then the temporarily path will be removed. The `acceptConnection` is defined in `IVC.hs`

More generally, the ls firstly posts its information in the path as a request, then rs response by write things to the path, then ls checked the information and confirmed the connection.

## clientServerConnection
Different thing is the `callback` on the server side and the use of `xsWatch`. So, the server will keep watching, and acting on the response.