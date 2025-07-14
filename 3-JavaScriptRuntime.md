# JavaScript Runtime

> Written by @linrongbin16, first created at 2024-08-10, last updated at 2024-09-11.

This RFC describes [JavaScript](https://en.wikipedia.org/wiki/JavaScript)/[TypeScript](https://www.typescriptlang.org/) and the JS runtime embedded in the RSVIM.

## Motivation

Scripting plays a most important role in (Neo)VIM editor: it drives the editor's looking and behavior, schedules background jobs, communicates with remote process, etc. It turns the editor into a script interpreter/runtime/virtual machine.

[Vim](https://www.vim.org/) uses [vimscript](https://www.vim.org/scripts/), which has very limited language features and performance, and documentation is quite rough. While [Neovim](https://neovim.io/) uses [lua5.1](https://www.lua.org/) (running on [LuaJIT](https://luajit.org/)) for much richer language features and great performance. It's so obvious and no need to prove, with a more powerful script language, the community can develop much more feature-rich plugins and provide more functions close to the modern text editors (such as [VsCode](https://code.visualstudio.com/)) and even IDE editing experiences.

Introducing third-party script languages is always a trend in (Neo)VIM's history. In the old days before Neovim appeared, people use [python](https://www.python.org/) and javascript (running on [node.js](https://nodejs.org/)) to achieve complex features in their plugins: code-complete, async code lint running in background, file explorer, etc. After Neovim brings lua, people still keep integrating third-party lua libraries for more features, such as async support via [luv](https://github.com/luvit/luv) (which brings a strong sense of separation when developing async scripts).

The whole reason is simply because: both of vimscript and lua lack of modern language features, and they don't have a modern package management tool like python's pip, node's npm. That's why RSVIM choose JavaScript, one of the most successful scripting languages. When comparing with js, lua has below shortcomings:

- It doesn't support many modern language features:
  - Async/await.
  - Functional programming.
- The community is not that active/popular:
  - The number of developers is not that large.
  - The open-sourced/third-party libraries are not that rich or widely used.
- [LuaRocks](https://luarocks.org/) (as lua's package manager) still has too many cross-platform compatibility issues on Windows. While [npm](https://www.npmjs.com/) (as js/ts package manager) is much more successful and popular.

However, js syntax can be really bad and chaotic (the success actually belongs to browsers, not the language itself). So the final target is scripting with TypeScript, while js plays the role of middle layer under the hood. Using ts brings even more benefits:

- More elegant and beautiful syntax designing.
- Static type system.
- Fully compatible with js community.

By introducing the js engine, RSVIM provides the best scripting environment that can interact with the editor, while also turns itself into a js/ts interpreter/runtime focused on editing.

## Runtime

The [V8](https://v8.dev/) engine is the best javascript engine widely used in many popular projects, RSVIM choose the same way to execute js scripts. But there are still many components needed to fill the gap between the final goal:

### Operations (OPs)

Operations extend js script's capabilities beyond the [ECMAScript](https://ecma-international.org/publications-and-standards/standards/ecma-262/) specification, since V8 engine strictly follows the ECMAScript guidelines, unable to:

1. Use system calls such as reading files, managing sockets, handling timers, etc.
2. Interact with RSVIM editor such as editing buffers, opening windows, changing colorschemes, etc.
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

Once we embed the package manager inside RSVIM editor just like [deno](https://deno.com/), all we need is sharing an example of config file with recommended plugins. In this way, both official and third-party plugins can be continuously rolled out and updated, the editor just needs to be a single executable file, responsible for providing an interface for js runtime and package management.

One step future, we could even directly integrate with javascript package registries such as [npm](https://www.npmjs.com/) and [jsr](https://jsr.io/).

### Conclusion

After all, RSVIM becomes a js runtime similar to [deno](https://deno.com/) in some ways, but only focus on editing and text processing, not for browsers or web development.

## References

- [The Internals of Deno](https://choubey.gitbook.io/internals-of-deno)
