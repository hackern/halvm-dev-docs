# Port

Port is HaLVM's way of calling Xen Event Channel.

The `toPort` and `fromPort` is wrapping/unwrapping a `Word16` type as a identifier.

`setPortHandler` is to install a function that will be invoked whenever the HALVM receives an event on the given channel. Basically a FFI wrapper for `events.h`'s `set_haskell_handler`

`unsetPortHandler` does a little more. It remove the current event handler on an event channel, returning it. This can be particularly useful when black-boxed components may need to chain together a series of event handlers. Calling this routine is safe even if no previous handler has been assigned to the port; in that case, the return value will be a noop.

`allocPort` allocate a fresh event channel that will be connected to by the given domain.

`bindRemotePort` binds another domain's port, which they've shared with us.

`bindVirq` bind virtual IRQ on virtual CPU to a port. This is a little strange. Since port, cpu and irq are all involved. The C function:

	evtchn_port_or_error_t
	xc_evtchn_bind_virq(xc_evtchn *xce, unsigned int virq)
	{
	return xce->ops->u.evtchn.bind_virq(xce, xce->ops_handle, virq);
	}

`allocUnboundPort`: Allocate a port on behalf of another domain, without causing the other end to bind it.

`sendOnPort`: Send an event on the given port.

`withPortMasked`: Perform the given operation within the IO monad with events on the given port disabled.

`bindPhysicalIRQ`: Bind a physical IRQ to an event channel. The arguments are the IRQ to map and a boolean stating whether or not the IRQ will be shared.

