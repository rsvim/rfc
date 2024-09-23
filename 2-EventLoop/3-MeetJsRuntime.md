# Meet Js Runtime

> Written by @linrongbin16, first created at 2024-09-23.

This RFC describe how the event loop should work along with javascript runtime.

## Limitation

When we introduce javascript engine, i.e. the V8 to RSVIM, a command line TUI application, everything changed because of it. There're several challenges:

- The `Isolate` of rust binding [rusty_v8](https://github.com/denoland/rusty_v8) is purely `!Send` and `Sync`, thus it doesn't work in a multi-thread environment.
- The conflicts between the start time minimization and user config file execution, and the data racing in a multi-thread environment.
- The plugin support, i.e. the multiple javascript files support.
- The development difficulty and learning curve on V8, which is the price of the most powerful and performant js engine.

## Single Thread

After some investigation and hesitation, the final event loop is designed to be: let the js runtime runs in the main thread, along with terminal keyboard/mouse events, and terminal rendering. Worker threads are only involved when they are suitable.

This is mostly because, the javascript language itself is designed to be running in single process, single thread, it doesn't support concepts such as thread and mutex (which is also a benefit for users when they're writing scripts because it's simple) at the very beginning. For the async, timeout and callbacks, they are handled by the tokio runtime's local tasks, i.e. the `spawn_local`, which should be actually a coroutine running on current thread.

Single thread brings determined and consistent behavior, and reduces the data syncing effort between multiple threads. For example users will never want to press `i` key and see the terminal still allow you press other keys but not go into the **INSERT** mode. Instead, users would rather block by the terminal and wait for it.

## Starting

The RSVIM itself becomes yet another js runtime actually, it will initialize the V8 engine first on starting. Then locate user config files, say `$XDG_CONFIG_HOME/rsvim/rsvim.{ts,js}`, and execute it, which is also running in a pure single thread. This design also makes the editor's behavior consistent, for example, if we let js runtime run in another thread, and start TUI application at the same time, user may find their editor config such as _indent size_, _tab space_ is first 8 bytes width (by default), then turns into 2 bytes (user config).

As a user config, the js file anyway need to be loaded first before user can operate on the TUI application.

## Built-in Types

Most famous js runtimes such as _Node.js_, _Deno_ and [_LLRT_](https://github.com/awslabs/llrt) are for general purpose, running web frameworks or applications on server side. The long history of javascript brings a lot of built-in APIs and specifications, which is quite a burden if we want to keep compatible with.

We could choose to directly use implement such as [`deno_core`](https://github.com/denoland/deno_core), which is actually great, it has:

1. Maybe the best js runtime framework written in rust.
2. Many built-in types implementations (i.e. the `JSON`, `decodeURI`, `Proxy`, `queueMicrotask` etc).

But the `deno_core` is designed for a general purpose js runtime running on server side, we don't want the `Deno` global object and many other web APIs to be built inside the RSVIM. Unless we have detailed understanding of every line of code in the entire codebase, it's still kind of out of control to embed it. For example the `console.log` API, when we implement it for RSVIM, it should never just print messages to `stdout`, instead, it should print messages in the command line inside the editor.

As a TUI editor, most APIs we want to provide is about the file system, IO, network, and IPC on local operating system. Manually implementing every API is more fit into the editor, and more controllable.
