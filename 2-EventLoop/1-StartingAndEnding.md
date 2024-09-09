# Starting and Ending

This RFC describes the starting and ending logic of the editor.

When the editor runs in tokio's multi-threaded and async event loop, we spawn async tasks to handle several things:

- IO:
  - File IO.
  - Network IO: pipe, named pipe, unix domain socket, tcp/udp, http(s), ssh, etc.
  - Terminal IO: user keyboard/mouse events. Note: _**stdout/stderr**_ printing is a sync IO operation that doesn't support writing in async mode.
- User js/ts scripts:
  - Callbacks: auto commands, delayed/timeout jobs, IO operations, child processes, etc. Note: _**IO operations**_ and _**child processes**_ should support both sync and async mode.
  - Js/ts modules (`require`/`import` keywords): modules will be loading and executed in async mode. Note: this will be in sync mode when editor running in headless/batch mode, because there's no non-blocking requirements when user doesn't interact with terminal.

Consider two special cases with above topics: starting and ending.

## Starting

The editor has several behaviors on start up:

1. TUI mode, allows user to interact with TUI and edit file contents.
2. Headless mode, uses as a js/ts runtime, without TUI and non-interactively with users.
3. Script mode, this mode uses the editor as a js/ts runtime, without TUI and non-interactively with users.

4. When there's no input files, creates a window and an empty buffer.
5. When there's 1 input file, creates a window, and 1 buffer for the input file, the buffer is binded on the created window.
6. When there's N (> 1) input files, creates a window, and N buffers for the N input files, but only the 1st file is binded on the created window.

The input files are loaded into corresponding buffers by an async task, with async file reading. During the file reading, user is still able to do normal operations inside the editor, for example: move the cursor, change to other editing modes, execute Vim commands, open new files, etc.

There are some edge cases worth to discuss:

### Buffer Synchronization between User Editing and File Loading

## Ending
