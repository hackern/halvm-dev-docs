# OtherDomain

> Various routines for doing things to other domains

Two flags about TLB are ported, `TLBEffect` and `TLBTarget`.

The comments in source are good as well.

In general, what a domain might do to other domain involves frames allocation, memory mapping, port (to bind) allocation, set memory limit etc.

The problem is that, why are these operations necessary?

Actually, these operations are merely defined, but never used in other files. They might be used when deugging, but I am not sure about these.