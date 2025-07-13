# Principles

> Written by @linrongbin16, created at 2024-03-16, last updated at 2025-07-13.

The RSVIM is still going to be the VIM editor, not anything else. It maintains almost the same product features as the VIM editor:

1. Terminal text editor, not GUI application.
2. Editing mode: normal, insert, visual/select, commandline, etc.
3. Core concepts such as window, buffer, commandline, statusline, tabline, etc.
4. Scripts and plugins.
5. And more to name.

But to re-invent VIM, instead of simply covnerting the project to another language, new designs/solutions will be introduced, old and out-dated things will be deprecated. Here're some topics come to me:

1. Use Rust as the developing language, as it has so much benefits on language features, performance and the great ecosystem.
2. Provide a consistent scripting runtime environment (or a virtual machine), with builtin support for:
   - Async/await.
   - Static typing system.
   - Package management.
3. Use Javascript/Typescript as the scripting language, as it's one of the most successful scripting languages, with powerful/performant JS/WASM engines and the great ecosystem.
4. Provide a powerful TUI engine that helps creating more friendly user interfaces and pretty rendering effects.
5. Provide an editing service, with builtin support for:
   - Remote editing.
   - Multiple clients working together.
6. Improve the project developing by leveraging existing community works:
   - Build better RSVIM user manual, API reference for Rust/Javascript/Typescript projects.
   - Improve code quality and performance with testing/benchmark frameworks.
   - Embed JS/WASM engines for Javascript/Typescript evaluation, and (maybe) integrate with NPM package manager.
