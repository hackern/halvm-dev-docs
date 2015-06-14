Core Code Analysis
===

## First Impression
Compared to conventional OS (or even House!), the `Core` focuses much more on the **hypervisor interfacing** and **communication between domain**.

I thought the main motivation is that, Xen have done a lot in managing the hardware resources, leave us a lot of APIs (like `SystemControl`, `DomainControl`, and `PhysicalDevice`) to call. Secondly, communication is the point as long as HaLVM might mostly be used on cloud service and might share a single physical machine with hundreds of other HaLVM.

Beyond that, there is also some "classical OS Concepts" in HaLVM `Core`, including:

* Memory Management (mostly deal with Xen memory scheme, like `MFN` and `Grants`)
* IRQ Processings  (in form of `Virqs`, but I didn't see its usage elsewhere)
* Basic IO Routines (Not seen application as well)

So, I will focus on these these parts:

1. Domain
2. Communication
3. Classical Concepts
4. Misc
