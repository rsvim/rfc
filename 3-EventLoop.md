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

## Task Queue

Main use cases of a VIM editor for async runtime are:

- Resolve filesystem (directories and file reading) for external scripts and plugins, and run them.
- Event subscription (in Vim editor it's called [auto commands](https://vimhelp.org/autocmd.txt.html#autocmd.txt)) and trigger callbacks.
- Timeout tasks.
- The `async` annotated javascript functions.

These use cases usually require we submit an async task (just like a function pointer with a context in c/c++ that literally allows us doing any logic) to a queue, schedule and run them later. Thus we would like a very general task queue inside the event loop, which can be selected along with crossterm's hardware events.

Rust doesn't support [dynamic dispatch akin to `virtual class` in c++](https://doc.rust-lang.org/book/ch17-01-what-is-oo.html#polymorphism), or at least it's not safe, which is not recommended and should be avoid in large-scale use. We may use the [`futures::stream::FuturesUnordered`](https://docs.rs/futures/latest/futures/stream/struct.FuturesUnordered.html) as a queue for all async tasks, i.e. the [`futures::future::Future`](https://docs.rs/futures/latest/futures/future/trait.Future.html) trait.
