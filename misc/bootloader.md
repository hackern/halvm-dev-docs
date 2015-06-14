# Bootloader

> This passage covers: `arch.h`, `elfnote.h`, `start.i386/x86_64.S`

## Arch
This is super simple, only defined `HALVM_STACK_SIZE` as 32 bytes. (seriously?)

But this seems not being used anywhere else.

## Elfnote
A bunch of comments in the source.

The main problem is:

1. Why is `elfnote` used instead of something else?
2. How is it related to the special aspect of HaLVM?

[for more details](http://www.cs.fsu.edu/~baker/devices/lxr/http/source/linux/include/linux/elfnote.h?v=2.6.25)

It is used in start assemblies combined with `xen/elfnote.h`.

> Various operating system vendors are shipping ELF binaries and those binaries expect different system call interfaces. In order to help operating systems correctly determine whether or not they can run an ELF program, and how to run it (e.g. what OS emulation to use), some operating system vendors have begun marking their ELF binaries with vendor-specific note elements, and placing those notes in the binaries' PT_NOTE sections.

## Start.S

> Following cruft taken rom Xen mini-os (which in turn stole it from linux guests, I think)
> (There was no copyright notice.)

There are just all kinds of gritty-nitty things, but if you'd like to learn the low-level details of running an OS on Xen, it is useful to refer to this [A tour with Mini-OS](http://wiki.xen.org/wiki/Mini-OS-DevNotes).

