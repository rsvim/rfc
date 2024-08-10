# Javascript Engine

> 2024-08-10

This RFC describes [Javascript](https://en.wikipedia.org/wiki/JavaScript)/[Typescript](https://www.typescriptlang.org/) and the JS engine embedded in the RSVIM.

## Motivition

Scripting plays the most important role in (Neo)VIM editor, it drives the editor's looking and behavior, schedule background job, communicate with remote process, etc. It turns the editor into a script interpreter/runtime/virtual machine.

[Vim](https://www.vim.org/) uses [vimscript](https://www.vim.org/scripts/), which has very limited language features and performance, and documentation is quite rough. While [Neovim](https://neovim.io/) uses [lua](https://www.lua.org/) (running on [LuaJIT](https://luajit.org/)) for much richer language features and great performance. It's so obvious and no need to prove, with a more powerful script language, the community can develop much more feature-rich plugins and provide more functions close to the modern text editors such as [Visual Studio Code](https://code.visualstudio.com/), even IDE editing experiences.

But when comparing with Javascript, one of the most successful programming languages, lua still lacks in below aspects:

- It doesn't support many modern language features:
  - Async/await.
  - Functional programming.
- [luarocks](https://luarocks.org/) (as lua's package manager) still has too many cross-platform compatibility issues on Windows. While [npm](https://www.npmjs.com/) (as js/ts package manager) is much more successful and popular.
- The most active community developers and rich third-party libraries.

However, Javascript's syntax can be really bad and chaotic (the success actually belongs the browsers instead of the language itself). So the final target is scripting with Typescript, while Javascript plays the middle layer under the hood. There're even more benefits:

- More elegant and beautiful syntax designing.
- Static type system.
- Fully compatible with Javascript's community.

By introducing the JS engine, RSVIM provides the best scripting environment that can interact with the editor.

## Architecture

The architecture of how Javascript interacts with Rust looks like:

```text
---The RSVIM instance----------------
|                                   |
|   ---API-----------------         |
|   |                     |         |
|   |                     |         |
|   |                     |         |
|   |                     |         |
|   -----------------------         |
|                                   |
|                                   |
|   ---Js Engine-----------         |              ---Js scripts/plugins-------
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   |                     |         |              |                          |
|   -----------------------         |              ----------------------------
|                                   |
|                                   |
|                                   |
|                                   |
------------------------------------|
```
