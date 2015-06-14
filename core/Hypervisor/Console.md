# Console

The core functions are:

1. initXenConsole
2. initConsole
3. readConsole
4. writeConsole

### General Model
The console is build on top of the Xen "event channel" and "MFN"(machine frame number").

There are also buffer set up for it.

An interesting thing is that, the messages are added to a TODO list like structure.

### Buffer
I/O buffer has size, which is 1024 and 2048 byte (I supposed). The interesting part is its declaration:

	outBuffer p = (#ptr struct xencons_interface, out) p

So, what is `#`, and what is `struct`? And is that some `(X, Y)` applied to the `p`?

According to `hsc2hs`, `#` will be identified in code generatior, for this case when `#` is followed by a `ptr`, it means to get a `pointer` of certain data type, or the `out` file of `struct xencons_interface` in C.

The result is a function `Storable b => Ptr a -> b -> IO ()`, when applied with a `p`, give us a curried function.

### "TODO"
First, `pullNewReads` check if there is any new data on the ring (the wrapper of a pointer), and if any, return the pulled string as well as a boolean indicating that (Maybe redundant?)

Then, there is `solveReadToDos`, which is interesting that, only the first pattern matches a condition in which there is any string returned. So, I thought the logic is that, it will process the TODOs and kept adding the solved string segmeng on the second argument for the tail call, and finally return the result after the TODOs are all completed.

The `solveWriteToDos` write the strings in the `WriteToDo` to the ring, and return `True` when its attempt actually succeeded.

### Console Thread
This piece of code is interesting, it starts a console session as a *thread*. In the input arguments, `commChan` is the `chan` provided by IVC module, while `ring` is actually a pointer, which is used to locate the data structure which is illustrated before.

	startConsoleThread :: Chan ConsoleMsg -> Ptr a -> Port -> IO ()
	startConsoleThread commChan ring port =
	  forkIO (runThread [] [] "") >> return ()
	 where
	  runThread rtodos wtodos rbuffer = do
	    msg                        <- readChan commChan
	    let (rtodos', wtodos')      = addNewToDos rtodos wtodos msg
	    (newreads, shouldSignalR)  <- pullNewReads ring
	    let rbuffer'                = rbuffer ++ newreads
	    (rtodos'', rbuffer'')      <- solveReadToDos rtodos' rbuffer'
	    (wtodos'', shouldSignalW)  <- solveWriteToDos wtodos' ring
	    when (shouldSignalR || shouldSignalW) $ sendOnPort port
	    runThread rtodos'' wtodos'' rbuffer''

Basically, this is a threads holding information about "Write TODO" and "Read TODO", as well as a buffer. The point is where this thread is launched? From the examples we have, I *supposed* that that there is only one outlet for console, i.e., dom0, so the `initXenConsole` is called by the child, to build a connection between `domU` and `dom0`.


### Read & Write Console

The `readConsole` read data from the console, blocking until the given number of bytes are avaiable.

The `writeConsole` write data to the console, blocking until the string has been written to the console driver. Note that writing to the console driver does not necessarily mean that the string has been flushed to whatever viewing device is attached to the console.