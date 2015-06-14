# libIVC

The `README` under the directory is good enough. But for a better format, I will simply copy them here.


This directory defines a C API for interacting with HaLVM domains via the
standard IVC mechanisms.

OVERVIEW OF IVC MECHANISMS
--------------------------

The HaLVM exports two different kinds of IVC, which can be further
parameterized by how the channels are set up. For all intents and
purposes, however, when designing IVC mechanisms you can consider
only the following possibilities:

### One-Way, Typed Channels
These channels always transfer the same kind of information 
over the channel in a single direction. For the purposes of
C programs, read this as always transferring the same type
(or struct) over the pipe, although the size of the type may
be variable in some instances.

### Two-Way, Typed Channels
Essentially a combination of two one-way channels using a single
memory page. The input and output channels can each send different
message types, but must only use one per direction. In other words,
the input channel can be strings while the output channel can be
32-bit signed integers, but you must always read strings from the
input channel and write integers to the output channel.

### Xen Device-Style Ring Buffers
At the most abstract level, these are arbitrary request-response
channels. This mechanism exactly matches the Xen protocol, and
this library merely exports the core routines and the rendezvous
functionality in the HaLVM.

IMPORTANT GENERAL CONSIDERATIONS
--------------------------------

When using the IVC mechanisms, little details in how objects are marshalled
can have dramatic effects. In general, the quickest way to get a Haskell
object out a pipe is to use the IVC library 'put' function and have the
generics support work out the detail. However, this will require more work
on the other side, reverse engineering how Haskell happens to dump that
object into memory. If you do choose this option, two hints: (1) strings are
dumped as ASCII strings of the format "%i %s" where the integer is the
length of the string and the string is the data being sent, and (2) numbers
tend to be put into memory in network order.

> For examples of these sorts of systems, see the ROT13Echo and SysLog examples.

The next level of effort, from the Haskell side, is to use the `Data.Binary`
type class to define exactly how the item should be stored in memory. By
doing so, how you should write your C structs should be considerably more
clear. On the Haskell side, please make sure of your byte ordering when
using the `Data.Binary` 'get' and 'put' functions on arbitrary types; in 
your own types or for unsigned numbers, you can use the relevant functions
in `Data.Binary.Get/Put` to force the order you prefer. 

Finally, on the C side, if your fields are not aligned -- or even if you're
just not sure -- you probably want to add the "packed" attribute to your
structures. This will forbid GCC from placing padding space between fields,
which can create havok with your wire protocols. To use, just add 
`__attribute__((__packed__))` after `struct` or `union`. If you have 
substructures, you'll need to pack them as well.

> For an example of this sort of system, see the CryptoDevice2 example.

Finally, this library exports the rendezvous portion of the standard Xen
device protocol, which the HaLVM also understands. Using this system can
be a bit of a pain compared to the previous two methods, but is a common
Xen standard and probably has better optimal performance. This system 
requires a bit more detailed effort on the part of the Haskell author, 
and some careful programming by the C author.

> For an example of this sort of system, see the CryptoDevice example.


KNOWN LIMITATIONS
-----------------

All IVC rendezvous in the current implementation of Xen are a three-stage
handshake with two actors, as follows:

    ACTOR A                         ACTOR B
    -------

    "My domain # is 4"

                                    "Hi #4, my name is B,
                                     here's a grant reference
                                     and event channel!"

    "Thanks, B!"

The Xen C libraries that Xen uses do not allow user code to create 
grant table entries; thus, users of this library cannot fulfil the
"Actor B" role above. For IVC channels, this is not much of a
problem. It does mean, however, that users of this library cannot be
device front ends. Sorry.

Finally, in order to make contact with the required kernel functionality,
programs using this library must be run as root.


C API
-----

I've tried to design the C API so that it is high-level and fairly
obvious, but just in case, here's a rundown of what's defined in the
header file:

	void initialize_libIVC_library();

This function must be called before invoking any other function in
the library. It sets up the communications paths to the xenstore
and xen kernel functions, so you really can't avoid it.

	in_chan *connect_one_way_in(char *name);
	out_chan *connect_one_way_out(char *name);
	inout_chan *connect_two_way(char *name);

Create an input, output, or bidirectional channel, respectively,
using "name" as a unique identifying key for rendezvous. It is
really very important that "name" not overlap with any other 
channel you're trying to rendezvous on.

	int read_inchan(in_chan *chan, void *buffer, int size);
	int read_chan(inout_chan *chan, void *buffer, int size);

Read at most `size` bytes from the input/bidirectional channel, 
placing the object in the given buffer, returning the number of
bytes read. Note that these channels do not provide for partial
reads; if you specify to read 4 bytes and the next object to
come on the wire is 12 bytes long, these routines will return 
0 (error). You will have to specify at least 12 to get the next
object.

	int read_unknown_inchan(in_chan *chan, void **buffer);
	int read_unknown_chan(in_chan *chan, void **buffer);

Reads in the next object, regardless of size, into a new buffer.
Places the address of this new buffer in `buffer`, and returns
the size of the read object. It is the responsibility of the 
caller to free the object when it is done with it.

	int write_outchan(out_chan *chan, void *buffer, int size);
	int write_chan(inout_chan *chan, void *buffer, int size);

Write the object in `buffer`, which is `size` bytes long, to the
given channel. Returns the number of bytes written; essentially,
0 on error or `size` on success.

	unsigned long inchannel_peer(in_chan *chan);
	unsigned long outchannel_peer(out_chan *chan);
	unsigned long channel_peer(inout_chan *chan);
    
Returns the domain ID of the domain on the other end of the channel.

	xen_backend *create_responder(char *name);

Create a new back-end device. This is somewhat similar to creating
a server socket in a sockets library. This creates the basic structures,
but in order to create connections you have to call all sorts of other
functions. (See also accept_connection). 

The name should be a unique name identifying the rendezvous point;
clients will use this same name to find the service, and other backends
and IVC channels *must* have different names.

	void accept_connection(xen_backend *, unsigned long *, void **, evtchn *);

Given a backend "server socket", block on the socket until someone
tries to connect to it, and then accept that connection attempt. The
last three arguments are all out arguments; accept will fill them in
with the domain ID of the other domain, a pointer to the shared memory
page, and the shared event channel, respectively. 


