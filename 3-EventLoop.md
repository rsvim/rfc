# Event Loop

> Written by @linrongbin16, 2024-09-03

This RFC describes the RSVIM's running loop.

## Top-down View

As mentioned in [TUI](https://github.com/rsvim/rfc/blob/e47afd180cc7038675addecf82efed040336ad72/1-TUI.md?#L9), the very basic running loop of RSVIM editor is just 3 steps:

1. Receive user keyboard/mouse events.
2. Handle user logic.
3. Render terminal or exit.

When such a classic running loop (for all GUI/TUI application) comes to terminal+rust, we specifically introduce:

- [Tokio](https://tokio.rs/) as asynchronize runtime.
- [Crossterm](https://github.com/crossterm-rs/crossterm) as hardware driver for terminal.

Tokio provides multiple infrastructures:

- Non-blocking event loop ([select](https://docs.rs/tokio/latest/tokio/macro.select.html)) on future streams:
  - TUI: Receive user keyboard/mouse events by crossterm's [event-stream feature](https://github.com/crossterm-rs/crossterm?tab=readme-ov-file#feature-flags).
  - IO: Read/write file system, network, tcp/http/ssh, etc.
  - Async: Schedule `async` annotated javascript functions inside scripts.
- Concurrent tasks schedule on multiple threadings:
  - Loading user scripts/plugins.
  - Syntax and colorscheme rendering.
  - Trigger auto-commands (event callbacks).
  - Schedule timeout or delayed tasks.

After all, RSVIM's event loop is similar to a javascript runtime like [node.js](https://nodejs.org/) or [deno](https://deno.com/), but focusing on text editing and TUI rendering.

## Context

The event loop is simply a global instance of data structure that contains everything inside the editor:

- UI widget tree (that contains the Vim windows, cursor, statusline, etc) and canvas.
- The Vim buffers.
- Editing mode.
- And more: javascript runtime, loaded scripts/plugins, etc.

## Editing Mode

[Editing mode](https://vimhelp.org/intro.txt.html#vim-modes) is managed by a finite-state machine, i.e. each mode is a state inside the editor:

```text
+----------------+
|                |
|     Start      |
|                |
+-------+--------+
        |
        |
        v
+----------------+               +------------------------+
|                |          i    |                        |
|  Normal Mode   +------+------->|  Insert Mode           |
|                |      |        |                        |
+----------------+      |        +-----------+------------+
        ^    ESC        |                    |
        +---------------+--------------------+
        |               |
        |               |        +------------------------+
        |               |   v    |                        |
        |               +------->|  CharWise-Visual Mode  |
        |               |        |                        |
        |               |        +-----------+------------+
        |    ESC        |                    |
        +---------------+--------------------+
        |               |
        |               |        +------------------------+
        |               |   V    |                        |
        |               +------->|  LineWise-Visual Mode  |
        |               |        |                        |
        |               |        +-----------+------------+
        |    ESC        |                    |
        +---------------+--------------------+
        |               |
        |               |        +------------------------+
        |               | CTRL-V |                        |
        |               +------->|  BlockWise-Visual Mode |
        |               |        |                        |
        |               |        +-----------+------------+
        |    ESC        |                    |
        +---------------+--------------------+
        |               |
        |               |        +------------------------+
        |               |  ...   |                        |
        |               +------->|  Other states/modes... |
        |               |        |                        |
        |               |        +-----------+------------+
        |    ESC        |                    |
        +---------------+--------------------+
        |               |
        |               |        +------------------------+
        |               |   :    |                        |
        |               +------->|  Command-Line Mode     |
        |                        |                        |
        |                        +-----------+-------+----+
        |    ESC                             |       |
        +------------------------------------+       |
                                                     |
+----------------+                                   |
|                |           qa!                     |
|      Exit      |<----------------------------------+
|                |
+----------------+
```

Each state can consumes the keyboard/mouse events and implements the corresponding behavior in editor.

## Task Queue

Main use cases of a VIM editor for async runtime are:

- Resolve filesystem (directories and file reading) for external scripts and plugins, and run them.
- Topic subscription and event consuming (in Vim editor it's called [auto commands](https://vimhelp.org/autocmd.txt.html#autocmd.txt), but can be treat as a more generally [publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)).
- Timeout tasks.
- The `async` annotated javascript functions and standard APIs provided by RSVIM editor.

These use cases usually require we submit a very general async task to a queue, schedule and run later, just like a function pointer with a context in c/c++ that literally allows us doing any logic. We also need the task queue be to a [`Stream`](https://docs.rs/futures/latest/futures/stream/trait.Stream.html), which can be selected along with crossterm's event stream. The [`FuturesUnordered`](https://docs.rs/futures/latest/futures/stream/struct.FuturesUnordered.html) can be the queue for all async tasks, i.e. the [`Future`](https://docs.rs/futures/latest/futures/future/trait.Future.html) trait.

Two more topics need to discuss:

1. The type of general async task.
2. Extreme and unlikely situations.

### General Async Task Type

Based on current design

### Extreme and Unlikely Situations

#### Cancel a Submitted Task

Simply clear the queue, this should not be a big deal.

#### Interrupt/Abort a Running Task

For example, when reading/writing a super big file, it can take minutes or even hours. It's dangerous if the read/write operation is interrupted without correctly open/close the file descriptor, which damages filesystem on storage device.

For such case, we have below choices:

1. Carefully insert manual checks on a global [`CancellationToken`](https://docs.rs/tokio-util/latest/tokio_util/sync/struct.CancellationToken.html) inside every async task, it notifies these running tasks to stop in safety.
2. Build up a pool to reserve these running tasks, and abort them when needed, while potential danger is at user's own risk.
3. Do nothing and simply wait for these running tasks complete.

The current behavior of Neovim and Vim is the option 3. We would also keep the compatible behavior, leave option 1 and 2 for future discussion if needed.
