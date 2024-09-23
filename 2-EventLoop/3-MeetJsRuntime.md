# Meet Js Runtime

> Written by @linrongbin16, first created at 2024-09-23.

This RFC describe how the event loop should work along with javascript runtime.

## Challenges

When we introduce javascript engine, i.e. the V8 to RSVIM, a command line TUI application, everything changed because of it. There're several challenges:

- The `Isolate` of rust binding [rusty_v8](https://github.com/denoland/rusty_v8) is purely `!Send` and `Sync`, thus it cannot work along with tokio runtime in a multi-thread environment.
- The conflicts between the start time minimization and user config file execution, and the data racing in a multi-thread environment.
- The plugin support, i.e. the multiple javascript files support.
- The development difficulty and learning curve on V8, which is the price of the most powerful and performant js engine.

## Single Thread

After some investigation and hesitation, the final event loop is designed to be: let the js runtime runs in the main thread, along with terminal keyboard/mouse events, and terminal rendering. Worker threads are only involved when they are suitable.

This is mostly because single thread natively brings the determined and consistent behavior for the editor, and reduces the data syncing effort between multiple threads. For example, users will never want to press `i` key and see the terminal still allowing you to press other keys but not go into the **INSERT** mode. Instead, users would rather blocked by the terminal and wait for it goes into **INSERT** mode.

The javascript language itself is also designed to be running in a single thread at the very beginning. It doesn't support concepts such as thread and mutex, while it is a benefit for users when writing scripts because the simplicity. For the async, timeout and callbacks inside js, they are handled by the tokio runtime's local tasks, i.e. the `spawn_local`, which should be actually a coroutine running on current thread.

## Starting

The RSVIM itself becomes yet another js runtime actually, it will initialize the V8 engine first on starting. Then locate user config files, say `$XDG_CONFIG_HOME/rsvim/rsvim.{ts,js}`, execute it, which is also running in a pure single thread. This design also makes the editor's behavior consistent. For example, if we let js runtime run in another thread, and start TUI application at the same time, user may find their configs such as _indent size_, _tab space_ is first 8 bytes width (default settings when editor start), then turns into 2 bytes (modified by executing the user config js running in another thread).

As a user config, the js file anyway need to be loaded first before user can operate on the TUI application.

Sequential configuration loading brings the start time, i.e. as users install more configs/plugins, the editor's startup time increases. The famous [lazy.nvim](https://github.com/folke/lazy.nvim) (Neovim's plugin manager) uses `VeryLazy` event to lazy load configs, and compile lua files into byte code to boost the start up time. For RSVIM, we may take a look at some solutions such as V8's snapshot, and js file bundle.

## Built-in Types

Most famous js runtimes such as _Node.js_, _Deno_ and [_LLRT_](https://github.com/awslabs/llrt) are for general purpose, running web frameworks or applications on server side. The long history of javascript brings a lot of web APIs and specifications, which is quite a burden if we want to keep compatible with.

We could choose to directly use implement such as [`deno_core`](https://github.com/denoland/deno_core), which is actually great, it has:

1. Maybe the best js runtime framework written in rust.
2. Many built-in types implementations (i.e. the `JSON`, `decodeURI`, `Proxy`, `queueMicrotask` etc).
3. It can resolves modules that marked in CDN, http(s), [deno.land](https://deno.land/std@0.224.0) and registries: [jsr](https://jsr.io/)/[npm](https://www.npmjs.com/) by detecting if exist, downloading and transpiling them on the fly.

But we also have reasons to not use it:

1. `deno_core` is designed for a general purpose js runtime running web frameworks and applications on server side, there is the `Deno` global object and many other web APIs to be built inside, but we may don't want them. For example the `console.log` API, when we implement it for RSVIM, it should never just print messages to `stdout`, instead, it should print messages in the command line inside the editor.
2. We don't want to implicitly download the plugins when starting the editor, simply resolve the plugins path on local file system should be good for RSVIM.
3. We are not 100% sure about the `deno_core` behavior, unless we have detailed understanding for every line of code in its entire codebase.

As a TUI editor, most APIs we want to provide is about the file system, IO, network, and IPC on local operating system, not for web applications. Manually implementing every API is more fit into the editor, and more controllable.
