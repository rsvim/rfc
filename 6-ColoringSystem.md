# Coloring System

> Written by @linrongbin16, first created at 2025-06-15.

This RFC describes an high-level overview for coloring system, include feature requirements, scope and techincal solutions.

## Background

Coloring system makes a text editor colorful. It contains two kinds:

1. The syntax color for source code text file of programming languages.
2. The UI widget color for rsvim editor.

### Syntax

Every programming language has its own syntax. For example:

- `c` language has keywords: `for`, `if`, `continue`, `goto`, etc. It has constants, string literals, variables, functions, etc.
- `c++` language has keywords: `new`/`delete`, `private`/`public` etc. It has classes/interfaces, members/methods, etc.
- `python` language has keywords: `def`, `len`, `and`/`or`, `filter`, etc.
- And more languages...

To let editors understand the source code, there is an syntax engine helps parse source code to tokens, i.e. tokenizer. This is similar to the compiler frontend, but syntax engines are optimized for speed, while compiler frontend are optimized for correctness.

In today's indrustry, there are actually only a few *popular* syntax engines:

- [TextMate](https://macromates.com/manual/en/language_grammars): The TextMate engine is a regex-based engine, created by the [textmate](https://github.com/textmate/textmate) editor. It is widely used by many editors: sublime-text (see [sublime-text syntax definition](https://www.sublimetext.com/docs/syntax.html#include-syntax)), vscode (see [vscode-textmate](https://github.com/microsoft/vscode-textmate)), atom (see [first-mate](https://github.com/atom/first-mate)), etc.
- [Treesitter](https://github.com/tree-sitter/tree-sitter): The treesitter engine is a parser-framework engine, it is actively maintained and also popular in many editors: helix (see [helix runtime files](https://github.com/helix-editor/helix/tree/master/runtime)), zed (see [zed extentions](https://github.com/zed-industries/zed/tree/main/extensions) and [zed languages crate](https://github.com/zed-industries/zed/tree/main/crates/languages/src)).

For each programming language, it has its own syntax config file, and needs an engine (inside the editor) to load/parse the config, thus the editor can scan the source code text file, and collect the tokens (i.e. keywords/variables/classes/functions) inside the file.

Vim (and Neovim) uses a self-implemented regex-based engine. Vscode/sublime-text uses the TextMate engine, which is also regex-based, but supports more advanced features. Zed/helix uses the treesitter engine.
