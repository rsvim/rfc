# Edit Files on Start Up

The editor accepts file names from command line [arguments](https://vimhelp.org/usr_02.txt.html#usr_02.txt), indicates the files should start to edit once on editor start up. When implementing this function with tokio event loop, we spawn an async task to handle the file IO and buffer creation. User first sees an empty window, then filled with buffer text contents once it's loaded from the input files. This never blocks the editor starting and removes the start up time.

But we need to consider more edge cases about the data race between file IO and user operations in a multi-threaded concurrent environment.
