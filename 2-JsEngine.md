# 2. Js Engine

This RFC describes the [Javascript](https://en.wikipedia.org/wiki/JavaScript)/[Typescript](https://www.typescriptlang.org/) and JS engine embedded in the RSVIM.

## Motivition

Scripting plays the most important role in (Neo)VIM editor, it drives the editor's behavior, automatically complete user's job, communicate with remote processes, etc. Any way, it's actually a script interpreter/runtime environment/virtual machine focused on editing. By introducing Javascript and JS engine, RSVIM provides the best scripting environment that can interact with the editor.

[Vim](https://www.vim.org/) uses [vimscript](https://www.vim.org/scripts/), which has very limited language features and performance, and documentation is quite rough. While [Neovim](https://neovim.io/) uses [lua](https://www.lua.org/) (running on [LuaJIT](https://luajit.org/)) for much richer language features and great performance. It's so obvious and no need to prove that, with a more powerful scripting language, the community can develop much more feature-rich plugins and provide more functions close to the modern text editors such as [Visual Studio Code](https://code.visualstudio.com/), even IDE editing experiences.

But comparing with Javascript/Typescript, one of the most successful programming languages, lua still lacks in below aspects:

- It doesn't support many modern language features:
  - Async/await.
  - Functional programming features such as lambda.
- - Static type system.
- [luarocks](https://luarocks.org/) (as lua's package manager) still has too many cross-platform compatibility issues on Windows. While [npm](https://www.npmjs.com/) (as js/ts package manager) is much more successful and popular.
