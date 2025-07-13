# Event Loop

> Written by @linrongbin16, first created at 2024-09-03, last updated at 2024-11-20.

This RFC describes the RSVIM's running loop.

## Running Loop

As mentioned in [RFC-1](https://github.com/rsvim/rfc/blob/e47afd180cc7038675addecf82efed040336ad72/1-TUI.md?#L9), the very basic running loop of RSVIM editor is just 3 steps:

1. Receive user keyboard/mouse events.
2. Handle user logic.
3. Render terminal or exit.

When such a (classic) running loop comes to terminal+rust, we specifically introduce:

- [Tokio](https://tokio.rs/) as async runtime.
- [Crossterm](https://github.com/crossterm-rs/crossterm) as hardware driver for terminal.

Tokio helps split the editor logic into more fine-grained tasks: blocking tasks and non-blocking tasks.

## Tasks and Non-Important Tasks

Notice we should not rashly say tokio helps us turning the editor into async. Because in this scenario, editing is always interacting with users.

### Blocking Tasks

For example, in rsvim edtior, when a user presses the key `i`, the editor goes to **Insert Mode**, then the user types some letters `a-z, A-Z` and numbers `0-9`, the editor appends these letters and numbers in the buffer, then the user presses the key `ESC`, the editor goes back to **Normal Mode**.

In this case, an editor waits for a user's action, finishes internal logic, renders the terminal, then waits for user's next action. This is the most important tasks for the editor, and the timeline should be always sync and blocking. Because user would rather wait for tasks done to get a deterministic and correct editor behavior.

### Non-Blocking Tasks

There are also a lot of tasks that provide better user experiences, while users don't want these tasks blocking the core editing functions, such as:

- Colorschemes.
- Background jobs.
- Multiple IPC/RPC connections.
- Child processes management.
- Source code token parsing.

We could leverage rust and tokio's async to dispatch these tasks, thus fully utilize all the CPU cores and never freeze the editor.

## Context

The context is a global data structure instance that contains all the data for the editor:

- UI widget tree (contains windows, cursor, statusline, etc) and canvas.
- Buffers.
- Editing mode.
- And more: javascript runtime, loaded scripts/plugins, etc.

It uses read/write locks for data synchronization across different threads.

## Editing Mode

[Editing mode](https://vimhelp.org/intro.txt.html#vim-modes) is managed by a finite-state machine, i.e. each mode is a state inside the editor:

![1](images/2-EventLoop.1.drawio.svg)

Each state can consumes the keyboard/mouse events and implements the corresponding behavior in editor.

## Async Task

Main use cases of a VIM editor for async runtime are:

- Resolve filesystem (directories and file reading) for external scripts and plugins, and run them.
- Topic subscription and event consuming (in Vim editor it's called [auto commands](https://vimhelp.org/autocmd.txt.html#autocmd.txt), but can be treat as a more generally [publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)).
- Timeout tasks.
- The `async` annotated javascript functions and standard APIs provided by RSVIM editor.

Let's consider some very extreme and unlikely situations:

### Cancel a Submitted Task

To clear all the submitted tasks, simply place a [CancellationToken](https://docs.rs/tokio-util/latest/tokio_util/sync/struct.CancellationToken.html) at the beginning of each task, check if it's cancelled before running. This should not be a big deal.

### Interrupt/Abort a Running Task

For example, when writing a super big file that takes minutes or even hours, it's dangerous if the write operation is interrupted without correctly close the file descriptor, which damages filesystem on storage device.

For such case, we have to wait for them complete to keep safe.
