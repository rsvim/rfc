# Meet Js Runtime

> Written by @linrongbin16, first created at 2024-09-23, last updated at 2024-09-27.

This RFC describe how the event loop should work when it meets the javascript engine.

## Challenges

When we introduce javascript engine, i.e. the V8 to RSVIM, a command line TUI application, everything changes because of it:

- The V8 [`Isolate`](https://docs.rs/v8/latest/v8/struct.Isolate.html) is `!Send` and `Sync`, it cannot work along with tokio runtime in a multi-thread environment.
- The conflicts between the requirement that minimizing the start time and user config file's execution, and the data racing in a multi-thread environment.
- The plugin support, i.e. how to support the structure of multiple javascript files.
- The development difficulty and learning curve on V8, which is the price of the most powerful and performant js engine.

## Single Thread

After some investigation and hesitation, the final event loop is designed to be still single thread, i.e. let the js runtime runs in the main thread along with terminal's receiving keyboard/mouse events and rendering. Worker threads are only involved when they are suitable.

This is mostly because single thread natively brings a consistent editor behavior, and avoids data racing issue between multiple threads. For example, user will never want to press `i` key and see the terminal still allowing you to press any other keys but not go into the **INSERT** mode. Instead, users would rather be blocked by the terminal and wait for it goes into **INSERT** mode.

The javascript language itself is also designed to be running in a single thread at the very beginning. It doesn't support concepts such as thread and mutex (while on the other hand, it is a benefit for users when writing scripts because the simplicity). For the async, timeout and callbacks features, they are handled by the tokio runtime's local tasks, i.e. the `spawn_local`, which should be actually a coroutine running on current thread.

## Starting

The RSVIM itself becomes yet another js runtime, it will initialize the V8 engine first on starting. Then locate user config files (say `$XDG_CONFIG_HOME/rsvim/rsvim.{ts,js}` or `$HOME/.rsvim.{ts,js}`), execute it, which is also running in a pure single thread. This design also makes the editor's behavior consistent. For example, if we let js runtime run in another thread, and start TUI application at the same time, user may find their configs such as _indent size_, _tab space_ is first 8 bytes width (default settings when editor start), then turns into 2 bytes (modified by executing the user config js running in another thread).

As a user config, the js file anyway need to be loaded first before user can operate on the TUI application.

Since the config file is loading in sequential order, it increases the startup time as user installs more and more configs/plugins. The famous [lazy.nvim](https://github.com/folke/lazy.nvim) (Neovim's plugin manager) uses `VeryLazy` event to lazy load configs, and compile lua files into byte code to boost the start up time. For RSVIM, we may take a look at some solutions such as V8's snapshot, and js bundles.

## Implementation

Most famous js runtimes such as _Node.js_, _Deno_ and [_LLRT_](https://github.com/awslabs/llrt) are for general purpose, running web applications on server side. The long history of javascript brings a lot of web APIs and specifications, which is quite a burden if we want to keep compatible with. There are several options when we want to implement the js runtime for our editor:

1. Directly rely on [`deno_core`](https://github.com/denoland/deno_core). It should be the best js runtime framework written in rust (for now), it has many built-in web APIs implementations (`JSON`, `decodeURI`, `Proxy`, `queueMicrotask`, etc), it can resolve remote modules from registries such as npm, jrs and CDN/http(s), download and transpile them on the fly.
2. Use some other tiny js engines such as [rquickjs](https://github.com/DelSkayn/rquickjs), which could be easier to implement.
3. Use [rusty_v8](https://github.com/denoland/rusty_v8) library, and manually write everything else to fill the gap between js engine and js runtime.

### `deno_core`

Even `deno_core` looks so great, we don't choose it for several reasons:

1. `deno_core` is designed for a general purpose js runtime running web applications on server side, there is the `Deno` global object and many other web APIs to be built inside, but we may don't want them. For example the `console.log` API, when we implement it for RSVIM, it should never just print messages to `stdout`, instead, it should print messages in the command line inside the editor.
2. We may don't want to implicitly download the plugins when starting the editor, simply resolve the plugins path on local file system should be good for an editor. Note: This possibility simply looks too powerful for an editor, we may leave it for future discuss.
3. We are not 100% sure about the `deno_core` codebase, unless we have detailed understanding for every line in its entire codebase, we cannot easily use it. Because js runtime framework is not like a json or random library, the API is quite clear and we just choose a popular one. We will have to understand the behavior overall and carefully work with it.

### Tiny Js Engines

The performance of QuickJs is worse than LuaJIT, which is not meet our requirements. The goal of the script runtime should be, roughly speaking, keep even with LuaJIT.

### V8

So finally the only option is use V8, and manually fill all the gaps, which is way more controllable.
