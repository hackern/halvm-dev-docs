# Profiling HaLVMs
> Copied from [https://github.com/GaloisInc/HaLVM/wiki/Profiling]()


Commit `42be44c6` brings initial support for profiling HaLVMs. Using this facility requires some understanding of normal GHC profiling behavior, combined with some special nonsense that only the HaLVM can provide.

## Why the nonsense?

As you read this document, the following question will probably occur to you: Why is this so hard?

Here is your answer:

GHC's profiling system uses a series of file system calls to dump information gained during the profiling run. In the normal world of GHC, this is fine, because there's a file system available on all its supported platforms. However, this is not true of the HaLVM, because "always having a file system around" would break two of the major goals of the HaLVM: (1) Do as much as possible in Haskell, and (2) allow HaLVM users to "opt out" of as many operating system features as possible. Thus, not having a file system available for profiling is a feature, not a bug.

Thus, to implement profiling, we have to go back on our promise a little bit. This involved re-writing a Xen virtual disk driver in C, amongst other evil things, and is why many of the steps below are more delicate than they seem like they should be.

Of course, if you'd like to clean this up, please feel free to jump in!

## Building with Profiling

The HaLVM, and all its associated base libraries, are built with profiling versions. You will need to build any libraries and executables with profiling support, as well, using the appropriate flags to `halvm-cabal` or `halvm-ghc`. Please see GHC and cabal's documentation for more information.

One note: if you want to profile a library, and you are building using Cabal, you'll need to find some way to add "-auto-all" to the Cabal build commands. This is a GHC-wide issue, not a HaLVM issue.

## Profiling a Virtual Machine

Classic GHC supports two types of profiling:

1. Classic time profiling, triggered by using the runtime flag `-p`, which generates a file named `Executable.prof`.
1. Heap profiling, triggered by using the runtime flag `-hc`, which generates a file named `Executable.hp`.

The HaLVM supports both of these options, along with any other GHC profiling flag. However, in order for the HaLVM to generate the appropriate data, you must provide it with disks on which to place the data. **You must use the disk name `xvdp1` for the `.prof` file, and `xvdp2` for the `.hp` file.** Unfortunately, because these are provided to the HaLVM as disks, you're going to have to guess how much space you're going to need for the files. `Prof` files are typically ASCII text, and will likely fit within a few megabytes. `HP` files, however, are collections of samples across the runtime of the program, so you'll have to make an intelligent guess based on how long you want to run the program.

You need to create the raw disk file like this example:

`dd if=/dev/zero of=prof.disk bs=1 count=1 seek=64M`

Once you have the files created, you can pass the runtime system flags via Xen's `extra` config field, and the disks in as you would any other disk. For example, if you wanted to collect both bits of profiling information, and had created the files "prof.disk" and "hp.disk", you might end up with a config file that looks like this:

     kernel = "my_halvm"
     memory = 64
     extra  = "+RTS -p -hc"
     disk   = [ "prof.disk,raw,xvdp1,rw", "hp.disk,raw,xvdp2,rw" ]

You should then be able to run your HaLVM normally.

## Fetching and processing profiling data

Once your HaLVM has completed, you now have two disks with profiling data in them. To get the data from the disk, the HaLVM now includes a handy program called `convert-profile`. Assuming our running example from the previous section, we could then execute the following commands to get the normal profile files:

     % convert-profile prof.disk my_halvm.prof
     % convert-profile hp.disk my_halvm.hp

Note that the former file only will be correctly generated if your HaLVM exits cleanly. If it crashes, then you will not get any timing information. You will likely get some heap profiling data, but it may lack data from near the end of the run.