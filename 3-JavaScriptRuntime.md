# JavaScript Runtime

> Written by @linrongbin16, first created at 2024-08-10, last updated at 2024-09-11.

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

- Both vimscript/lua are not modern script languages, and not popular compared with python/javascript.
- Both Vim/Neovim doesn't have a package manager, they only detect scripts and evaluate them. Not like python's pip, node's npm, etc.

That's why Rsvim choose JavaScript, one of the most successful scripting languages. However, js syntax can be really bad and chaotic (the success actually belongs to browsers, not the language itself). So the final target is scripting with TypeScript, while js plays the role of middle layer under the hood. Using ts brings even more benefits:

- More elegant and beautiful syntax designing.
- Static type system.
- Fully compatible with js community.

By introducing the js engine, Rsvim provides the best scripting environment that can interact with the editor, while also turns itself into a js/ts interpreter/runtime focused on editing.

## Runtime

The [V8](https://v8.dev/) js engine is the best javascript engine widely used in many popular projects, Rsvim choose the same way to execute js scripts. But there are still many components needed to fill the gap between the final goal:

### JavaScript APIs (Operations)

For most general script programming languages such as python, they provide builtin [types](https://docs.python.org/3/library/stdtypes.html)/[functions](https://docs.python.org/3/library/functions.html) and [standard library](https://docs.python.org/3/library/index.html). Javascript runtime is similar, the difference is: js engine is provided by a third-party library (i.e. V8 js engine). A js engine covers the [ECMA-262](https://ecma-international.org/publications-and-standards/standards/ecma-262/) standard, while js runtimes provide the whole standard library and fill in many other gaps, which includes:

- [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API): Most popular javascript-based runtimes share a compatible implementations today.
- Specific APIs: Different runtimes provide have their own specific APIs, i.e. browsers such Chrome/Firefox provide the `document` DOM tree APIs, server-side runtimes such as node/deno provide their own APIs to manage the operating systems.

Operations extend js script's capabilities beyond the [ECMAScript](https://ecma-international.org/publications-and-standards/standards/ecma-262/) specification, since V8 engine strictly follows the ECMAScript guidelines, unable to:

1. Use system calls such as reading files, managing sockets, handling timers, etc.
2. Interact with Rsvim editor such as editing buffers, opening windows, changing colorschemes, etc.
3. Provide a standard library (similar to [Node.js Official APIs](https://nodejs.org/docs/latest/api/documentation.html) and [Deno Standard Library](https://deno.land/std)) to improving developing efficiency such as string manipulation, filesystem, path, streams, etc.

### TypeScript Support

V8 engine is exclusively designed to run javascript code and does not support TypeScript. To address this, typescript code must be translated into js through a transformation process, enabling V8 engine to execute it.

### Load Module

Modules are resolved by `require` and `import` keywords, thus allow user to create multiple file structured configs, and load third-party plugins.

### Package Management

Both (Neo)Vim ship a lot of builtin scripts/plugins with their releases, which provide a lot of builtin APIs and functionalities. But this method has two shortcomings:

1. Some scripts are rapidly developed and changed, which are not suitable for release in a fixed time manner.
2. Users are forced to install them, even there exists better alternative plugins in community.

All of them point to the core problem: (Neo)Vim doesn't have its own package management system.

Once we embed the package manager inside Rsvim editor just like [deno](https://deno.com/), all we need is sharing an example of config file with recommended plugins. In this way, both official and third-party plugins can be continuously rolled out and updated, the editor just needs to be a single executable file, responsible for providing an interface for js runtime and package management.

One step future, we could even directly integrate with javascript package registries such as [npm](https://www.npmjs.com/) and [jsr](https://jsr.io/).

### Conclusion

After all, Rsvim becomes a js runtime similar to [deno](https://deno.com/) in some ways, but only focus on editing and text processing, not for browsers or web development.

## References

- [The Internals of Deno](https://choubey.gitbook.io/internals-of-deno)
