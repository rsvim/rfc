# Starting and Ending

This RFC describes the starting and ending logic of the editor.

When the editor runs in tokio event loop, we spawn async tasks to handle several things:

- IO:
  - File IO.
  - Network IO: pipe, named pipe, unix domain socket, tcp/udp, http(s), ssh, etc.
  - Terminal IO: user keyboard/mouse events. Note: _**stdout/stderr**_ printing is a sync IO operation that doesn't support writing in async mode.
- User js/ts scripts:
  - Callbacks: auto commands, delayed/timeout jobs, IO operations, child processes, etc. Note: _**IO operations**_ and _**child processes**_ should support both sync and async mode.
  - Js/ts modules (`require`/`import` keywords): modules will be loading and executed in async mode. Note: this will be in sync mode when editor running in headless/batch mode, because there's no non-blocking requirements when user doesn't interact with terminal.

the file IO and buffer creation. User first sees an empty window, then filled with buffer text contents once it's loaded from the input files. This never blocks the editor starting and removes the start up time.

But we need to consider more edge cases about the data race between file IO and user operations in a multi-threaded concurrent environment.

## Description

As mentioned in [RFC-4](https://github.com/rsvim/rfc/blob/main/4-WindowsAndBuffers.md), the editor has several behaviors:

1. Creates a window and an empty buffer if no input files specified (by default).
2. Creates a window, and N buffers for N input files, only the 1st input file is binded on the current window.
