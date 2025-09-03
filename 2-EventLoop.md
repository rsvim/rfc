# Event Loop

> Written by @linrongbin16, first created at 2024-09-03, last updated at 2025-09-03.

This RFC describes the Rsvim's running loop.

## Running Loop

As mentioned in [RFC-1](https://github.com/rsvim/rfc/blob/main/1-TUI.md), the very basic running loop of Rsvim editor is just 3 steps:

1. Receive user keyboard/mouse events.
2. Handle user logic.
3. Render terminal or exit.

![1](images/1-TUI.1.drawio.svg)

When such a (classic) running loop comes to terminal+rust, we specifically introduce:

- [Tokio](https://tokio.rs/) as async runtime.
- [Crossterm](https://github.com/crossterm-rs/crossterm) as hardware driver for terminal.

Tokio helps split the editor logic into more fine-grained tasks: blocking tasks and non-blocking tasks. Notice we should not rashly say tokio helps us turning the editor into async. Because in this scenario, i.e. text editing, the editor is always interacting with users.

For example, when a user execute below operations:

1. Press key `i` - the editor goes to **Insert Mode**.
2. Types letters `a-z` and `A-Z` and numbers `0-9`, the editor inserts these letters and numbers in the buffer.
3. Press key `ESC`, the editor goes back to **Normal Mode**.

In these operations, the **Interaction** should always be synchronous, i.e. it should follows "input" => "calculation" => "output" for each user's operation, thus achieve a consistent application behavior. Even some operations will block the TUI, this should still stay synchronous, because consistent behavior has highest priority.

You may ask: then what does tokio do? and how do we benefit from tokio's async tasks? - Because in a modern text editor, there are too many low-level tasks/services running together to provide users with a very comfortable editing experience. Only a few core operations (i.e. text editing) should always stay synchronous, other tasks can be asynchronous, for example:

- Source code syntax analysis and highlighting.
- LSP services management.
- LSP services management.

## Starting

When rsvim starts, it follows below steps to initialize

## Running

### Blocking/Sync Tasks

For example, in rsvim edtior, when a user presses the key `i`, the editor goes to **Insert Mode**, then the user types some letters `a-z, A-Z` and numbers `0-9`, the editor appends these letters and numbers in the buffer, then the user presses the key `ESC`, the editor goes back to **Normal Mode**.

In this case, an editor waits for a user's action, finishes internal logic, renders the terminal, then waits for user's next action. This is the most important tasks for the editor, and the timeline should be always sync and blocking. Because user would rather wait for tasks done to get a deterministic and correct editor behavior.

### Non-Blocking/Async Tasks

There are also a lot of tasks that provide better user experiences, while users don't want these tasks blocking the core editing functions, such as:

- Colorschemes.
- Background jobs.
- Multiple IPC/RPC connections.
- Child processes management.
- Source code analysis (token parsing).

We could leverage rust and tokio's async to dispatch these tasks, thus fully utilize all the CPU cores and never freeze the editor.

Tokio's async task has an extra benefit because it can be used to implement JavaScript's `Promise` and `async`/`await` keyword. Each time user's scripts use the `Promise` and `async`/`await`, the V8 js engine will stops running and gives the CPU back to the editor, hold scripts logic until next event loop.

This is exactly what all JavaScript-based runtimes ([node.js](https://nodejs.org/), [deno](https://deno.com/)) do.

## Exiting

### Interruptible Tasks

Consider when an editor is going to exit, async tasks can be split into two types: interruptible (cancellable) and non-interruptible (non-cancellable).

Most read-only tasks, such as colorschemes updating, they are interruptible, the editor can just abandon them and exit.

### Non-Interruptible Tasks

For write file tasks, or other resources related tasks (IPC/RPC, child-processes management), it is dangerous if editor forcibly interrupts them, which can damages users data and services.

For such case, we have to wait for them complete to keep safe.
