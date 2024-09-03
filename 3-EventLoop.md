# Event Loop

> Written by @linrongbin16, 2024-09-03

This RFC describes the RSVIM's running loop.

## Top-Down View

As mentioned in [TUI](https://github.com/rsvim/rfc/blob/e47afd180cc7038675addecf82efed040336ad72/1-TUI.md?#L9), the very basic running loop of RSVIM editor is just 3 steps:

1. Receive user keyboard/mouse events.
2. Handle user logic.
3. Render terminal or exit.

When such a classic running loop (for all GUI/TUI application) comes to terminal+rust, we specifically introduce:

- [Tokio](https://tokio.rs/) as asynchronize runtime.
- [Crossterm](https://github.com/crossterm-rs/crossterm) as hardware driver for terminal.

Tokio builds multiple infrastructures:

- Non-blocking event loop that loop ([select](https://docs.rs/tokio/latest/tokio/macro.select.html)) on future streams:
  - TUI: Receive user keyboard/mouse events by crossterm's [event-stream feature](https://github.com/crossterm-rs/crossterm?tab=readme-ov-file#feature-flags).
  - IO: Read/write file system, network, tcp/http/ssh, etc.
- Multiple threads that concurrently schedule (include delayed) tasks.
