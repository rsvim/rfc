# Config File

This RFC describes the config file, i.e. the `.rsvimrc.js` or `.rsvimrc.ts` config for the editor.

## JavaScript Engine

The [V8](https://v8.dev/) engine is the best javascript engine nowadays, which RSVIM editor embeds to execute js scripts just like many other projects. But there are still many components needed to fill the gap between the final goal:

- Operations (OPs): Extend config's capabilities beyond the [ECMAScript](https://ecma-international.org/publications-and-standards/standards/ecma-262/) specification, since V8 engine strictly follows the ECMAScript guidelines, unable to perform real-world tasks like reading files, managing sockets, handling timers, etc.
- TSC/SWC: V8 engine is exclusively designed to run js code and does not support TypeScript. To address this, ts code must be translated into js through a transformation process, enabling V8 engine to execute it.
- Load module: Resolve modules to enable `require` and `import` keywords, which are used to load other scripts and third-party installed plugins.
- Package (plugins) manager: Manage third-party plugins and leverage existing js package registry [npm](https://www.npmjs.com/) and [jsr](https://jsr.io/).

## References

- [The Internals of Deno](https://choubey.gitbook.io/internals-of-deno)
