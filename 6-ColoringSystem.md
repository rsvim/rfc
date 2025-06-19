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
- [TreeSitter](https://github.com/tree-sitter/tree-sitter): The TreeSitter engine is a parser-framework engine. Different from the regex-based engine, it actually implements each parser for each programming language. It is actively maintained and also popular in many editors: helix (see [helix runtime files](https://github.com/helix-editor/helix/tree/master/runtime)), zed (see [zed extensions](https://github.com/zed-industries/zed/tree/main/extensions) and [zed languages crate](https://github.com/zed-industries/zed/tree/main/crates/languages/src)).

The syntax engine is directly built inside the editor, while it has a separate syntax config file that defines how to parse the language source code.

- For vim, it embeds all the syntax configs in its [runtime/syntax](https://github.com/vim/vim/tree/master/runtime/syntax) folder. Every language has its `.vim` syntax file, for example [c.vim](https://github.com/vim/vim/blob/master/runtime/syntax/c.vim), [cpp.vim](https://github.com/vim/vim/blob/master/runtime/syntax/cpp.vim), [python.vim](https://github.com/vim/vim/blob/master/runtime/syntax/python.vim).
- For sublime-text, it has the [sublimehq/Packages](https://github.com/sublimehq/Packages). Each language has its `.sublime-syntax` config file, for example [C.sublime-syntax](https://github.com/sublimehq/Packages/blob/master/C%2B%2B/C.sublime-syntax), [C++.sublime-syntax](https://github.com/sublimehq/Packages/blob/master/C%2B%2B/C%2B%2B.sublime-syntax), [Python.sublime-syntax](https://github.com/sublimehq/Packages/blob/master/Python/Python.sublime-syntax).
- For vscode, it embeds the [extensions](https://github.com/microsoft/vscode/tree/main/extensions) folder. Each language has its `.tmLanguage.json` config file, for example [c.tmLanguage.json](https://github.com/microsoft/vscode/blob/main/extensions/cpp/syntaxes/c.tmLanguage.json), [cpp.tmLanguage.json](https://github.com/microsoft/vscode/blob/main/extensions/cpp/syntaxes/cpp.tmLanguage.json), [python.tmLanguage.json](https://github.com/microsoft/vscode/blob/main/extensions/python/syntaxes/MagicPython.tmLanguage.json).
- For TreeSitter (zed/helix), it has a community to maintain [a list of parsers](https://github.com/tree-sitter/tree-sitter/wiki/List-of-parsers) for each programming languages. Each language has its parser.

A syntax config helps the editor detect the source code text file by the file type/extension, i.e. it maps the file type to its syntax definition.

Regex-based engines (vim, textmate) syntax files are mostly config files like json/yaml/xml. While TreeSitter parser is a `parser.c` that implements the tokenizer parser for the language, and it needs to compile (with C/C++ compiler) into dynamical library (`.so`, `.dylib`, `.dll`) and load into the editor to work with TreeSitter.

Once source code text file are parsed into tokens, these syntax information are also been used by other functions/features of the editor. For example textobjects (vim's [textobjects](https://vimhelp.org/motion.txt.html#text-objects), helix's [textobjects](https://docs.helix-editor.com/textobjects.html)), indents, and other syntax related features.

### Theme

Once editors parsed the tokens from a source code text file, it needs another config to give these tokens different colors to make it colorful. Here comes the theme config (vim calls it colorscheme), and editors usually allow users to customize their themes.

- For vim, it embeds some default colorschemes in its [runtime/colors](https://github.com/vim/vim/tree/master/runtime/colors) folder.
- For vscode, it embeds some default themes in its [extensions](https://github.com/microsoft/vscode/tree/main/extensions) folder, sub-folders with `theme-` prefix.
- For helix, it embeds TreeSitter queries in its [runtime/queries](https://github.com/helix-editor/helix/tree/master/runtime/queries) folder (each language also needs a `query` file to define how editors can query the tokens from TreeSitter), themes in its [runtime/themes](https://github.com/helix-editor/helix/tree/master/runtime/themes).

A theme config file actually maps each token to its color (RGB, css name, terminal ansi code) and visual effects (underline, bold, italic).

Vim/Neovim editors also have other colors, not only syntax colors for source code text files. For example Neovim has multiple highlighting groups for **floating-window**: [NormalFloat](https://neovim.io/doc/user/syntax.html#hl-NormalFloat), [FloatBorder](https://neovim.io/doc/user/syntax.html#hl-FloatBorder), [FloatTitle](https://neovim.io/doc/user/syntax.html#hl-FloatTitle), etc.

Such kind of colors are not related to programming language syntaxes, but more related for UI widgets.

### Ecosystem

All the popular editors have their community to continuously contribute to the syntaxes and themes. This requrie a lot of time and efforts.

- Vim: Programming languages will maintain a `.vim` syntax file, to help developers to use their language with Vim/Neovim editors.
- Sublime-text: Programming languages will maintain a `.sublime-syntax` syntax file, to help developers to use their language with sublime-text editor (and all the editors/viewers compatible with it).
- VsCode: Programming languages will maintain a `.tmLanguage.json` syntax file, to help developers to use their language with vscode editor (and all the editors/viewers compatible with it).
- TreeSitter: Many programming languages will maintain its `parser.c` parser, to help developers to use their language with TreeSitter embedded editors.

Compared the syntax configs and parsers, only creating a syntax engine and specifications is easy, but building a whole community for all the popular programming languages is hard. It may takes many years.

## Engine

I believe rsvim should choose a ready-to-work syntax engine, leverage the existing community, thus to archive a quick win. Apparently we only have below choices:

1. Vim regex-based syntax engine.
2. Sublime-text/vscode TextMate syntax engine.
3. TreeSitter syntax engine.

### Vim Regex-Based Engine

Pros:

1. Has great performance. Based on some benchmark, it is the fastest, but also support least features, anyway it works.
2. Can directly use all existing vim's syntax files. It can help rsvim inherits the syntaxes/colorshemes from Vim community.

Cons:

1. Vim's syntax/colorscheme is written in `vim` script, which is completely not compatible with rsvim. Unless we create a vimscript interpreter, or a tool to convert vim scripts into js scripts. Both solutions seems not easy/possible, because it will lead to create a vimscript interpreter with rust. Such ideas are been tried several times by the community, and finally failed, please see [libvim](https://github.com/onivim/libvim).

### TextMate

Pros:

1. Has great performance, slower than Vim's regex-based engine, but provide more accurate tokens (it support syntax scope).
2. Popular and widely used by many editors.
3. TextMate has the rust implementation [syntect](https://github.com/trishume/syntect).
4. Both vim's syntax engine and TextMate are regex-based, and they have a error-tolerant result when parsing partial of the source code text file, which helps a lot for performance. For example, when user opens a very big source code file (500 MB), and let's say the syntax engine needs 3 seconds to parse the while source code text, it can block the user editing. And on every user insertion/deletion operation, it also make the editor laggy/slow. But we can improve the speed by parsing only part of the source code text, the part that shows in the window instead of the whole file. Even the parsing result is not 100% correct, the results can still match some keywords/string literals/variables, and give some colors. This is a trade-off between correctness and performance. As a visual dessert for the eyes, this features/services can be downgraded in exchange for better performance.

Cons:

1. The TextMate community seems slowly dying. Because TextMate itself is first created by the [textmate](https://github.com/textmate/textmate) editor (a open source project from Apple). Today textmate is used by sublime-text and vscode. Sublime-text is close source software so we don't know how it implements the textmate engine. Vscode implements the textmate engine in [vscode-textmate](https://github.com/microsoft/vscode-textmate), which is written in javascript. It seems when an editor wants to use textmate engine, it will have to maintain its own textmate implementations and syntax specifications.
2. If rsvim directly uses `syntect` library, it means rsvim will have to use the `.sublime-syntax` config files, or the `.tmLanguage` config files. The biggest `.sublime-syntax` configs are maintained by sublime-text's open source packages: [sublimehq/Packages](https://github.com/sublimehq/Packages), and the biggest `.tmLanguage` configs are maintained by github linguist grammars [github-linguist/linguist/vendor/grammars](https://github.com/github-linguist/linguist/tree/main/vendor/grammars) (most these grammars are not updated for many years).
3. If rsvim directly uses `vscode-textmate` library, it means rsvim will have to implement a javascript-runtime to run the library. Which performance can be a big issue (syntax engine should be directly embedded inside editor and be very performant).

### TreeSitter

Pros:

1. Most accurate parsing results/tokens. Note: We don't consider LSP servers as a syntax engine here, even it also provide semantic tokens.
2. TreeSitter has an official library and rust binding. The library is actively maintained, and it has an actively maintained community for most programming languages, see the [list of parsers](https://github.com/tree-sitter/tree-sitter/wiki/List-of-parsers).
3. TreeSitter supports incremental parsing, it should be performant on every user editing.

Cons:

1. TreeSitter is slower than regex-based engine, especially on super big files.
2. TreeSitter parsers need to be compiled (with C/C++ compiler) into dynamical library (`.so`, `.dylib`, `.dll`) on user's local machine, then load into the editor to work with TreeSitter. Note: a collection of pre-built parsers can alleviate the need for C/C++ compilers in some popular OS (Windows/Linux/MacOs) and CPU architectures (x86_64/amd64/arm64).

## Solution

The final solution choice is: TextMate vs TreeSitter.

I believe TreeSitter is a better choice because both itself and its community are actively maintained, it also has a clear documentation. The issues for rsvim are:

1. How to alleviate the needs for C/C++ compilers to help users avoid compiling the parsers.
2. How to avoid the slow speed of parsing the whole file when user first opens a source code text file.
3. How to make the syntaxes pluggable with rsvim editor. As rsvim is designed with a strong concept of being as a javascript-runtime, all plugins are actually packages that can be installed/removed/upgraded by a package management tool (just like `node` and `npm`), thus the `rsvim` binary will not embed any plugins inside itself. While we will provide some **official** plugins for rsvim and users can choose not to install them, if community has create better plugins, which is a common case for Vim editors.

### Avoid Local Compilation

Since a parser for a language in TreeSitter is a `parser.c` file, and it needs to be compiled into a dynamical library with C/C++ compiler on user's local machine. We can provide a plugin for rsvim to collect/prebuild/download all the syntax parsers for user.

TreeSitter team also plans to support `wasm` library, to let parsers work with javascript engines such as `v8`, to avoid C dynamical library.

### Avoid Blocking On First Open

When user first opens a file and TreeSitter parses the whole file, it can be slow and block user.

We can use the `async` operation provided by **Tokio** runtime, the async operation is scheduled in a separate thread, and avoid blocking. But in the meanwhile, there will be no syntaxes available.

### Pluggable Syntaxes

Image we create a `syntaxes.rsvim` project (written in javascript/typescript) as an official plugin for rsvim, it is highly similar to the [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) plugin (for Neovim). Once user install it, it will try to automatically download the prebuilt dynamical library (or download the parser and build/compile it with C/C++ compiler) for user when user first opens a source code text file. And it also provide several commands to let user install/upgrade/remove parsers by themself.

We can use GitHub Actions to automatically upgrade/build all the language parsers by running a daily job to upgrade them, and release the prebuilt dynamical libraries in GitHub Releases for some most popular OS: Windows/Linux x86_64, MacOS arm64. For other OS, user will have to install a C/C++ compiler to build the parsers by themself.
