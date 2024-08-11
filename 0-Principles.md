# Principles

> Written by @linrongbin16, 2024-03-16

The RSVIM is going to still be the VIM editor, not something else. It maintains almost the same product features as the VIM editor:

1. Terminal executable, not GUI application.
2. VIM editing mode.
3. UI components such as window, buffer, commandline, statusline, tabline, etc.
4. Scripting/plugin system.
5. More to name.

But to re-invent VIM, we introduce new things, and deprecate old things. Here're some topics to discuss:

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
