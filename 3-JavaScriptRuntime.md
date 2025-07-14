# JavaScript Runtime

> Written by @linrongbin16, first created at 2024-08-10, last updated at 2025-07-14.

This RFC describes [JavaScript](https://en.wikipedia.org/wiki/JavaScript)/[TypeScript](https://www.typescriptlang.org/) and the JS runtime embedded in the Rsvim.

## Motivation

Scripting plays a most important role in (Neo)VIM editor: it drives the editor's looking and behavior, schedules background jobs, communicates with remote process, etc. It turns the editor into a script interpreter/runtime/virtual machine.

[Vim](https://www.vim.org/) uses [vimscript](https://www.vim.org/scripts/), which has very limited language features and performance, and documentation is quite rough. While [Neovim](https://neovim.io/) uses [lua5.1](https://www.lua.org/) (running on [LuaJIT](https://luajit.org/)) for much richer language features and great performance. It's so obvious and no need to prove, with a more powerful script language, the community can develop much more feature-rich plugins and provide more functions close to the modern text editors (such as [VsCode](https://code.visualstudio.com/)) and even IDE editing experiences.

Introducing third-party script languages is always a trend in (Neo)VIM's history. In the old days before Neovim appeared, people use [python](https://www.python.org/) and javascript (running on [node.js](https://nodejs.org/)) to achieve complex features in their plugins: code-complete, linter/formatter, file explorer, etc. After Neovim brings lua, people still keep integrating third-party lua libraries for more features, such as [`vim.uv`](https://neovim.io/doc/user/lua.html#vim.uv) for async support, [`vim.iter`](https://neovim.io/doc/user/lua.html#vim.iter), etc.

There are many issues:

- Vimscript is indeed the best DSL designed for Vim editor, most syntax are designed for Vim, for example `:(n/i/v)map`, `:set`. But when community try to develop more heavy plugins, it doesn't handle the use cases, compared with other general programming languages.
- `vim.uv` (luv/libuv) has a big gap with Vim's original event loop that cannot be bridged, plugin developers have to call [`vim.schedule()`](<https://neovim.io/doc/user/lua.html#vim.schedule()>) to manually switch between the event loop. Because Vim's internal event loop was not designed for lua, new libuv event loop is been wrapped around Vim's event loop, instead of replace it (usually one app should only has one event loop).
- Vim/Neovim embed many builtin plugins, users don't have a choice to uninstall them, and cannot get timely updates until next release. For Vim it is not a big issue because Vim actually uses a rolling release policy, but for Neovim it is not good since Neovim releases by year or quarter.

If we think of Vim/Neovim editor as a scripting language interpreter, the reason of all these issues is simple:

- Both vimscript/lua are not modern script languages, and not that popular compared with python/javascript.
- Both Vim/Neovim doesn't have a package manager, they only detect scripts and evaluate them. Not like python's pip, node's npm, etc.

That's why Rsvim choose JavaScript, one of the most successful scripting languages. However, js syntax can be really bad and chaotic (the success actually belongs to browsers, not the language itself). So the final target is scripting with TypeScript, while js plays the role of middle layer under the hood. Using ts brings even more benefits:

- More elegant and beautiful syntax designing.
- Static type system.
- Fully compatible with js community.

By introducing the js engine, Rsvim provides the best scripting environment that can interact with the editor, while also turns itself into a js/ts interpreter/runtime focused on editing.

## Runtime

The [V8](https://v8.dev/) js engine is the best javascript engine widely used in many popular projects, Rsvim choose the same way to execute js scripts. But there are still many components needed to fill the gap between the final goal:

### JavaScript APIs

For most general script programming languages such as python, they provide builtin [types](https://docs.python.org/3/library/stdtypes.html)/[functions](https://docs.python.org/3/library/functions.html) and [standard library](https://docs.python.org/3/library/index.html). JavaScript runtime is similar, the difference is: js engine is provided by a third-party library (i.e. V8 js engine). A js engine covers the [ECMA-262](https://ecma-international.org/publications-and-standards/standards/ecma-262/) standard, while js runtimes provide the whole standard library and fill in many other gaps, which includes:

- [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API): Most popular javascript-based runtimes share a compatible implementations today.
- Specific APIs: Different runtimes provide have their own specific APIs.
  - For browsers such Chrome/Firefox, they provide extra APIs such as `document` DOM tree.
  - For server-side runtimes such as Node/Deno, they provide their own APIs: [Node APIs](https://nodejs.org/api/n-api.html), [Deno APIs](https://docs.deno.com/api/deno/~/Deno).
  - For Rsvim editor, we need to provide many Vim related APIs such as windows, buffers, key mappings, commands, etc.

### TypeScript Support

V8 engine is exclusively designed to run JavaScript code and does not support TypeScript. To address this, ts code must be transformed into js, thus enables V8 engine to evaluate it.

### Load Module

Modules are resolved by `require` and `import` keywords, thus allow user to create multiple file structured configs, and load third-party plugins.

### Package Management

Image Rsvim directly supports [npm](https://www.npmjs.com/) packages, users can simply install plugins with below steps:

1. Specify either `package.json` file in Rsvim config directory `~/.rsvim` or `$XDG_CONFIG_HOME/rsvim`.
2. Run `npm i` to install all packages and their dependencies via npm registry. Run `npm up` to upgrade, `npm rm` to remove as well.
3. Open Rsvim editor, it just correctly detects all the npm packages as Rsvim plugins.

In this way, Rsvim can handle plugins with a new way:

- There is no builtin plugins, they are separated from the `rsvim` executable. Users can choose whether to install/uninstall/upgrade a plugin.
- Some _must have_ plugins will be supported by Rsvim team as the _official_ plugins.
- Plugin developers can upload their plugins to npm registry, and users can get timely updates from the registry, user don't have to wait for next release for an _official_ plugin updates.

### Conclusion

After all, Rsvim becomes a specialized javascript-based runtime similar to node/deno, but it focus on editing and text processing, not browsers or web.
