# Coloring System

> Written by @linrongbin16, first created at 2025-06-15.

This RFC describes an high-level overview for coloring system, include feature requirements, scope and technical solutions.

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

In today's indrustry, there are actually only a few _popular_ syntax engines:

- [Vim's syntax engine](https://github.com/vim/vim/blob/master/src/syntax.c): Vim implements a regex-based syntax engine by itself. It is only used by Vim/Neovim.
- [TextMate](https://macromates.com/manual/en/language_grammars): The TextMate engine is also a regex-based engine, created by the [textmate](https://github.com/textmate/textmate) editor. It is widely used by many editors: sublime-text (see [sublime-text syntax definition](https://www.sublimetext.com/docs/syntax.html#include-syntax)), vscode (see [vscode-textmate](https://github.com/microsoft/vscode-textmate)), atom (see [first-mate](https://github.com/atom/first-mate)), etc.
- [Treesitter](https://github.com/tree-sitter/tree-sitter): The treesitter engine is a parser-framework engine. Different from the regex-based engine, it actually implements each parser for each programming language. It is actively maintained and also popular in many editors: helix (see [helix runtime files](https://github.com/helix-editor/helix/tree/master/runtime)), zed (see [zed extensions](https://github.com/zed-industries/zed/tree/main/extensions) and [zed languages crate](https://github.com/zed-industries/zed/tree/main/crates/languages/src)).

The syntax engine is directly built inside the editor, while it has a separate syntax config file that defines how to parse the language source code.

- For vim, it embeds all the syntax configs in its [runtime/syntax](https://github.com/vim/vim/tree/master/runtime/syntax) folder. Every language has its `.vim` syntax file, for example [c.vim](https://github.com/vim/vim/blob/master/runtime/syntax/c.vim), [cpp.vim](https://github.com/vim/vim/blob/master/runtime/syntax/cpp.vim), [python.vim](https://github.com/vim/vim/blob/master/runtime/syntax/python.vim).
- For sublime-text, it has the [sublimehq/Packages](https://github.com/sublimehq/Packages). Each language has its `.sublime-syntax` config file, for example [C.sublime-syntax](https://github.com/sublimehq/Packages/blob/master/C%2B%2B/C.sublime-syntax), [C++.sublime-syntax](https://github.com/sublimehq/Packages/blob/master/C%2B%2B/C%2B%2B.sublime-syntax), [Python.sublime-syntax](https://github.com/sublimehq/Packages/blob/master/Python/Python.sublime-syntax).
- For vscode, it embeds the [extensions](https://github.com/microsoft/vscode/tree/main/extensions) folder. Each language has its `.tmLanguage.json` config file, for example [c.tmLanguage.json](https://github.com/microsoft/vscode/blob/main/extensions/cpp/syntaxes/c.tmLanguage.json), [cpp.tmLanguage.json](https://github.com/microsoft/vscode/blob/main/extensions/cpp/syntaxes/cpp.tmLanguage.json), [python.tmLanguage.json](https://github.com/microsoft/vscode/blob/main/extensions/python/syntaxes/MagicPython.tmLanguage.json).
- For treesitter (zed/helix), it has a community to maintain [a list of parsers](https://github.com/tree-sitter/tree-sitter/wiki/List-of-parsers) for each programming languages. Each language has its parser.

A syntax config helps the editor detect the source code text file by the file type/extension, i.e. it maps the file type to its syntax definition.

Regex-based engines (vim, textmate) syntax files are mostly config files like json/yaml/xml. While treesitter parser is a `parser.c` that implements the tokenizer parser for the language, and it needs to compile (with C/C++ compiler) into dynamical library (`.so`, `.dylib`, `.dll`) and load into the editor to work with treesitter.

Once source code text file are parsed into tokens, these syntax information are also been used by other functions/features of the editor. For example textobjects (vim's [textobjects](https://vimhelp.org/motion.txt.html#text-objects), helix's [textobjects](https://docs.helix-editor.com/textobjects.html)), indents, and other syntax related features.

### Theme

Once editors parsed the tokens from a source code text file, it needs another config to give these tokens different colors to make it colorful. Here comes the theme config (vim calls it colorscheme), and editors usually allow users to customize their themes.

- For vim, it embeds some default colorschemes in its [runtime/colors](https://github.com/vim/vim/tree/master/runtime/colors) folder.
- For vscode, it embeds some default themes in its [extensions](https://github.com/microsoft/vscode/tree/main/extensions) folder, sub-folders with `theme-` prefix.
- For helix, it embeds treesitter queries in its [runtime/queries](https://github.com/helix-editor/helix/tree/master/runtime/queries) folder (each language also needs a `query` file to define how editors can query the tokens from treesitter), themes in its [runtime/themes](https://github.com/helix-editor/helix/tree/master/runtime/themes).

A theme config file actually maps each token to its color (RGB, css name, terminal ansi code) and visual effects (underline, bold, italic).

Vim/Neovim editors also have other colors, not only syntax colors for source code text files. For example Neovim has multiple highlighting groups for **floating-window**: [NormalFloat](https://neovim.io/doc/user/syntax.html#hl-NormalFloat), [FloatBorder](https://neovim.io/doc/user/syntax.html#hl-FloatBorder), [FloatTitle](https://neovim.io/doc/user/syntax.html#hl-FloatTitle), etc.

Such kind of colors are not related to programming language syntaxes, but more related for UI widgets.

### Ecosystem

All the popular editors have their community to continuously contribute to the syntaxes and themes. This requrie a lot of time and efforts.

- Vim: Programming languages will maintain a `.vim` syntax file, to help developers to use their language with Vim/Neovim editors.
- Sublime-text: Programming languages will maintain a `.sublime-syntax` syntax file, to help developers to use their language with sublime-text editor (and all the editors/viewers compatible with it).
- VsCode: Programming languages will maintain a `.tmLanguage.json` syntax file, to help developers to use their language with vscode editor (and all the editors/viewers compatible with it).
- Treesitter: Many programming languages will maintain its `parser.c` parser, to help developers to use their language with treesitter embedded editors.

Compared the syntax configs and parsers, only creating a syntax engine and specifications is easy, but building a whole community for all the popular programming languages is hard. It may takes many years.

I believe rsvim should choose a ready-to-work syntax engine, leverage the existing community, thus to archive a quick win.

## Engine Solutions

Apparently we only have below choices:

1. Vim regex-based syntax engine.
2. Sublime-text/vscode TextMate syntax engine.
3. Treesitter syntax engine.

### Vim Regex-Based Engine

Pros:

1. Can directly use all existing vim's syntax files.

### TextMate

### Treesitter
