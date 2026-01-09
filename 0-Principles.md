# Principles

> Written by @linrongbin16, created at 2024-03-16, last updated at 2025-07-13.

The Rsvim is still going to be the VIM editor, not anything else. It maintains almost the same product features as the VIM editor:

1. Terminal text editor, not GUI application.
2. Editing mode: normal, insert, visual/select, commandline, etc.
3. Core concepts such as window, buffer, commandline, statusline, tabline, etc.
4. Scripts and plugins.
5. And more to name.

But to re-invent VIM, instead of simply converting it into another programming language, new designs/solutions will be introduced, old/out-dated things will be deprecated. Here are some topics:

1. Use rust as the developing language, as it has so many benefits and there is no need to list them one by one.
2. Provide a consistent scripting runtime environment (or say, language interpreter, virtual machine), with modern programming language features:
   - Async/await.
   - Static typing system.
   - Package management.
3. Use JavaScript/TypeScript as the script language, as it's one of the most successful script languages, with powerful V8 js engine and the great ecosystem.
4. Provide a powerful TUI engine that helps creating more friendly user interfaces and pretty rendering effects.
5. Provide an editing service, with builtin support for:
   - Remote editing.
   - Multiple clients working together.
6. Improve project quality and overall experience:
   - Build better user manuals and API references document.
   - Improve code quality and performance with testing/benchmark frameworks.
   - Integration with NPM package manager. It will be great if users can specify their `package.json` for all the JavaScript/TypeScript written plugins (install with `npm i`, upgrade with `npm up`, remove with `npm rm`), and rsvim editor can correctly load all the packages as plugins.
