# Event Loop

> Written by @linrongbin16, first created at 2024-09-03, last updated at 2024-11-20.

This RFC describes the RSVIM's running loop.

## Running Loop

As mentioned in [RFC-1](https://github.com/rsvim/rfc/blob/e47afd180cc7038675addecf82efed040336ad72/1-TUI.md?#L9), the very basic running loop of RSVIM editor is just 3 steps:

1. Receive user keyboard/mouse events.
2. Handle user logic.
3. Render terminal or exit.

When such a (classic) running loop comes to terminal+rust, we specifically introduce:

- [Tokio](https://tokio.rs/) as asynchronize runtime.
- [Crossterm](https://github.com/crossterm-rs/crossterm) as hardware driver for terminal.

Tokio runtime turns the running loop from sync to async, i.e. the main thread only handles keyboard/mouse events and renders to terminal, all the other laggy jobs are spawned with async tasks running in multi-threaded environment and sync data back to editor and update UI after finished. Here are some examples:

- IO:
  - File IO.
  - IPC/RPC: Pipe, named pipe, unix domain socket, tcp/udp, http(s), ssh, etc.
  - Terminal: Keyboard/mouse events. Note: the sync _**stdout/stderr**_ operation is still used for rendering terminal.
- Callbacks:
  - Delayed/timeout jobs.
  - Auto commands on Vim events.
  - File watcher.
- (User) js/ts scripts:
  - The `async` annotated functions and `Promise` functions.
  - The `require` and `import` keywords (js modules).
- Heavy CPU or big memory block workload:
  - Syntax and colorscheme rendering.
  - Text object.
  - Token parsing.

In this way, RSVIM's running loop is actually similar to [deno](https://deno.com/), but focusing on text editing and TUI rendering.

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
