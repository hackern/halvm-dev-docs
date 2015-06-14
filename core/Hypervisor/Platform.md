# Platform

This file specifies *various routines for interacting with the underlying hardware platform*.

The comments are informative so I won't write a lot here.

In general, this are another layer of wrapping combing `domainControlOp`, `systemControlOp` and `physicalControlOp`, i.e., an abstraction over the Xen mechanism.

So, supposing that you want to port it to the bare metal, this layer and above should be almost untouched.