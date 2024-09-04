# Javascript Engine

> Written by @linrongbin16, 2024-08-10

This RFC describes [Javascript](https://en.wikipedia.org/wiki/JavaScript)/[Typescript](https://www.typescriptlang.org/) and the JS engine embedded in the RSVIM.

## Motivition

Scripting plays the most important role in (Neo)VIM editor, it drives the editor's looking and behavior, schedule background job, communicate with remote process, etc. It turns the editor into a script interpreter/runtime/virtual machine.

[Vim](https://www.vim.org/) uses [vimscript](https://www.vim.org/scripts/), which has very limited language features and performance, and documentation is quite rough. While [Neovim](https://neovim.io/) uses [lua](https://www.lua.org/) (running on [LuaJIT](https://luajit.org/)) for much richer language features and great performance. It's so obvious and no need to prove, with a more powerful script language, the community can develop much more feature-rich plugins and provide more functions close to the modern text editors such as [Visual Studio Code](https://code.visualstudio.com/), even IDE editing experiences.

Introducing third-party scripting languages is always a trend in (Neo)VIM's history. In the old days before Neovim appeared, people use [python](https://www.python.org/) and javascript on [node.js](https://nodejs.org/) to achieve more complex features in their plugins: code-complete, async code lint running in background, etc. After Neovim brings lua, people still keep integrating third-party lua libraries for more features, such as async support via [luv](https://github.com/luvit/luv) (which brings a strong sense of separation when developing async scripts).

The whole reason is simply because: both of vimscript and lua still lack many modern language features to achieve the real power. That's why RSVIM choose Javascript, one of the most successful scripting languages. When comparing with js, lua still lacks in below aspects:

- It doesn't support many modern language features:
  - Async/await.
  - Functional programming.
- The community is not that active/popular:
  - The number of developers is not that large.
  - The open-sourced/third-party libraries are not that rich or widely used.
- [luarocks](https://luarocks.org/) (as lua's package manager) still has too many cross-platform compatibility issues on Windows. While [npm](https://www.npmjs.com/) (as js/ts package manager) is much more successful and popular.

However, js syntax can be really bad and chaotic (the success actually belongs to browsers, not the language itself). So the final target is scripting with Typescript, while js plays the role of middle layer under the hood. Using ts brings even more benefits:

- More elegant and beautiful syntax designing.
- Static type system.
- Fully compatible with js community.

By introducing the js engine, RSVIM provides the best scripting environment that can interact with the editor, while also turns itself into a js/ts interpreter/runtime focused on editing.

## Architecture

The architecture of how Javascript interacts with Rust looks like:

```text
+---RSVIM-------------------------------------------------------+
|                                                               |
|                                                               |
|   +--API-----------------+      +--Editor-Function-------+    |
|   |                      |      |                        |    |
|   |   `vim` global var   +--+-->|  Option/var/win/buf    |    |
|   |                      |  |   |                        |    |
|   +----------------------+  |   +------------------------+    |
|              ^              |                                 |
|              |              |                                 |
|              |              |   +--Task-Queue------------+    |
|              |              |   |                        |    |
|              |              +-->|  Callback/timer/async  |    |
|              |              |   |                        |    |
|              |              |   +------------------------+    |
|              |              |                                 |
|              |              |                                 |
|              |              |   +--Garbage-Collection-----+   |
|              |              |   |                         |   |
|              |              +-->|  Heap Objects           |   |
|              |              |   |                         |   |
|              |              |   +-------------------------+   |
|              |              |                                 |
|              |              |                                 |
|              |              |   +--Misc-------------------+   |
|              |              |   |                         |   |
|              |              +-->|  Terminal/Child process |   |
|              |                  |                         |   |
|              |                  +-------------------------+   |
|              |                                                |
|              |                                                |
|   +--Js-Engin+-----------+      +--Ts-Compiler------------+   |    +--Js/Ts-Scripts----------+
|   |                      |      |                         |   |    |                         |
|   |  Option/var/win/buf  |<-----+  Compile Ts to Js       |<--+----+  Configs and plugins    |
|   |                      |      |                         |   |    |                         |
|   +----------------------+      +-------------------------+   |    +-------------------------+
|                                                               |
|                                                               |
+---------------------------------------------------------------+
```

## Interaction

Interaction between rust and javascript requires two sides: rust side and javascript side.

For rust side:

1. It loads external javascripts
2. It executes javascript code (via the engine).

For javascript side:

1. It uses [ECMA scripts](https://ecma-international.org/publications-and-standards/standards/ecma-262/) to literally implement anything.
2. It uses editor APIs provided by RSVIM to drive the editor instance.
3. (Optionally) it uses very limited part of standard library ported from other runtimes to improve developing efficiency, while web and browser related APIs are not supported. For example:

   - [Node.js Official APIs](https://nodejs.org/docs/latest/api/documentation.html)
   - [Deno Standard Library](https://deno.land/std)

## Package Management
