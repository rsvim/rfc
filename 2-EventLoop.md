# Event Loop

> Written by @linrongbin16, first created at 2024-09-03, last updated at 2025-09-04.

This RFC describes the Rsvim's running loop. Also recommend reading [RFC-3.2](3-JavaScriptRuntime/2-ImportModules.md).

## Running Loop

As mentioned in [RFC-1](1-TUI.md), the very basic running loop of Rsvim editor is just 3 steps:

1. Receive terminal keyboard/mouse events.
2. Handle user logic.
3. Render terminal or exit.

![1](images/1-TUI.1.drawio.svg)

When such a running loop comes to rust, we specifically introduce:

- [Tokio](https://tokio.rs/) as async runtime.
- [Crossterm](https://github.com/crossterm-rs/crossterm) as hardware driver for terminal.

Tokio helps split the editor logic into more fine-grained tasks: blocking tasks and non-blocking tasks. Notice we should not rashly say tokio helps us turning the editor into async. Because in this scenario, i.e. text editing, the editor is always interacting with users.

For example, when a user execute below operations:

1. Press key `i`: editor goes to **Insert Mode**.
2. Types `a-z`/`A-Z` and `0-9`: editor inserts these letters and numbers in the buffer.
3. Press key `ESC`: editor goes back to **Normal Mode**.

In these operations, the **process** should always be synchronous, i.e. it follows "input" => "calculation" => "output" for each operation, thus achieve a consistent behavior. Even some operations will block the TUI, this should still stay synchronous, because consistent behavior always has the highest priority.

You may ask: then what will tokio do? and how do we benefit from tokio's async tasks? - Because in a modern text editor, there are too many low-level tasks/services running together to provide a comfortable editing experience. Only a few core operations (i.e. above operations) should always stay synchronous, other tasks can run asynchronously or in parallel, for example:

- Syntax analysis and highlighting.
- LSP services management.
- Child processes management.
- IPC/RPC connections.
- And a lot more...

> If all these calculation logic run synchronously, you will stuck on opening every file/buffer, or even simply typing every character.

You will find most UI rendering effects and analysis tasks has a low priority, even they provide a much better user experience, but the service quality can be downgraded or cancelled. We spawn these tasks with tokio's async runtime, thus to not block core text editing, their calculation results will be sync back once they are done, or just be abandoned.

For example, the syntax analysis task will be run every time user insert a new character. A new syntax analyzing task is spawned to tokio. In the meanwhile, there may already have some ongoing syntax analyzing tasks calculating, once a newest task is spawned, all the old tasks can be cancelled, since their results are not useful any more. Or, user is exiting the editor, then all the ongoing analysis are no longer needed.

Thus all tasks inside rsvim can be split into two types: blocking tasks and non-blocking tasks.

### Blocking/Sync Tasks

Only a few core text editing logic always stays synchronous and blocking. Because user would rather wait for them done to get a consistent and correct behavior.

### Non-Blocking/Async Tasks

Many other logic can run asynchronously or in parallel, because user don't want to wait for them, thus fully utilize all the CPU cores and never freeze the editor.

Tokio's async task has an extra benefit because it can be used to implement JavaScript's `Promise` and `async`/`await` keyword. Each time user's scripts use the `Promise` and `async`/`await`, the V8 js engine stops running and gives the "main" thread to the editor, hold the context until next event loop. This is exactly what all JavaScript-based runtimes ([node.js](https://nodejs.org/), [deno](https://deno.com/)) do.

## Pseudo-Code Process

To make it more clear what rsvim is doing inside, here's a main loop process written with pseudo-code:

```text
1  Main:
       Reading arguments from CLI
       If arguments == "-V/--version" or "-h/--help":
           Print information and exit
5      Create data structures: BuffersManager, UIWidgetTree, JsRuntime, TaskTracker, etc.
       Initialize (execute) js configs.
       (Initialize buffers) If arguments provide file names:
            Create one buffer for each file name, reading file content into buffer. Set the first buffer as default buffer.
       Else:
10          Create a default buffer with no file name, empty content.
       Initialize a default window, binded with the default buffer.
       Initialize terminal into raw mode, and render TUI.
       Loop:
           `tokio::select!` on multiple streams asynchronously:
15             - `crossterm::event::EventStream`
               - Master Channel Receiver, if receives "Exit" message, break the loop
               - Js Channel Receiver
           Render TUI
19     Recover terminal, exit
```

Let's go through this line by line:

1. For line 1, it is the the entry of our program `rsvim`.
2. For line 2-4, it reads the arguments feed into `rsvim`. If arguments are `-V`/`--version` or `-h`/`--help`, it simply prints information and exit.
3. For line 5-12, the editor initialize 3 components:
   - Data structures such as buffers, UI tree, task tracker, etc.
   - Js runtime, include all V8 components.
   - Turn terminal into raw mode, and render for the first time. If the default buffer has file content, it will first show in the terminal.
4. For line 14-15, the editor starts to read from terminal input, i.e. user can start interacting with the editor. The loop uses `tokio::select!` to read from multiple streams asynchronously:
   - `crossterm::event::EventStream`: All user keyboard/mouse events are receiving through this stream.
   - Master channel receiver and Js channel receiver: Once master receives the "Exit" message, it breaks the loop.

   > NOTE: Tokio's runtime is multi-threaded and requires data structures to be `Arc` to keep thread safe. While V8 js engine is single-threaded and all data structures are `Rc`, which are non-thread safe. Thus rsvim introduces these 2 channels to send data to each other.

5. For line 16, once the event is been processed, and the editor renders the internal data changes to TUI.
6. For line 17, the editor recovers the terminal and exit the program.

As you can see, actually it still follows the "input" => "calculation" => "output" process.

## Exiting

### Interruptible Tasks

Consider when an editor is going to exit, async tasks can be split into two types: interruptible (cancellable) and non-interruptible (non-cancellable).

Most read-only tasks, such as colorschemes updating, they are interruptible, the editor can just abandon them and exit.

### Non-Interruptible Tasks

For write file tasks, or other resources related tasks (IPC/RPC, child-processes management), it is dangerous if editor forcibly interrupts them, which can damages users data and services.

For such case, we have to wait for them complete to keep safe.
