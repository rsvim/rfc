# Coloring and Syntax Parser

> Written by @linrongbin16, first created at 2025-06-15, last modified at 2026-01-09.

This RFC describes an high-level overview for coloring system and syntax parser, include feature requirements, scope and technical solutions.

## Introduction

Coloring system makes a text editor colorful. It contains multiple components to work together and provide a user experience:

1. A syntax parser that recognizes text semantics that can support programming languages, structured formats and even more potential formats.
2. A theme that defines a set of colors for all the parsed tokens.
3. A coloring painter that renders colors onto the text and finally print to canvas/terminal.

Syntax parser also serves as a very fundamental component that generates structured and tokenized information from a text file (mostly source code), which can help implementing many advanced features for an editor:

- Indent: detect how many spaces should use when starting the next new line when writing code.
- Text objects: detect structure boundaries such as inside a function, string literals, etc.
- Code outlining and folding: identify code blocks such as function, class, for/while loop, etc.
- Symbols: identify function, parameters, class, variables, constants, etc.
- Brace/Tag Matching: identify corresponding opening/closing braces or HTML/XML tags.

### Syntax Parser

Nowadays, there are actually only 2 kinds of syntax parsers:

1. Regex-based engines:
   - [Vim's syntax engine](https://github.com/vim/vim/blob/master/src/syntax.c): Vim implements a regex-based syntax engine by itself. It is only used by Vim/Neovim.
   - [TextMate](https://macromates.com/manual/en/language_grammars): The TextMate engine is first created by the [textmate](https://github.com/textmate/textmate) editor, now it is widely used by many editors: [sublime-text](https://www.sublimetext.com/docs/syntax.html#include-syntax), [vscode](https://github.com/microsoft/vscode-textmate), [atom](https://github.com/atom/first-mate), etc.
   - And some more...
2. [Tree-sitter](https://github.com/tree-sitter/tree-sitter): A tokenizer-based engine that parses source code more accurately, works more like a compiler. It is also widely used by many editors: [zed](https://zed.dev/blog/syntax-aware-editing), [helix](https://helix-editor.com/) and [GitHub](https://www.youtube.com/watch?v=a1rC79DHpmY).

> NOTE: We don't count [language-server-protocol](https://microsoft.github.io/language-server-protocol/) as syntax parser, while it can be a big improvement but not mandatory.

### Ecosystem

Two more components are important as well for a coloring system:

- Syntax rule for each different programming language. A language has its own rule/parser to define its own syntax, the definition is loaded by the syntax parser to tokenize the text files of that language, generates the structured/tokenized information for the text files that opened by an editor.
- Theme configure for each different themes (Vim/Neovim call it [colorscheme](https://vimhelp.org/syntax.txt.html#%3Acolorscheme)). A theme defines how it actually looks for each token of a text file, e.g. color (frontground, background) and visual effects (bold, italic, underline, etc) of the text.

These two components can have hundreds/thousands or more configurations, because a programming language can have a rule configuration, a theme/colorscheme can have a theme configuration. They are mostly maintained by the open source community due to the development effort, i.e. users participate in the development and testing of all these syntax rules and color themes.

### Pros & Cons

| Category                                       | Tree-sitter                                                                          | TextMate                                                                                                                                                                                                   | Vim Syntax Engine                                                                                 |
| ---------------------------------------------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Quality                                        | Most Accurate                                                                        | Fine                                                                                                                                                                                                       | Worst                                                                                             |
| Performance (especially on opening a new file) | Lowest                                                                               | Fine                                                                                                                                                                                                       | Highest                                                                                           |
| Incremental Parsing                            | Yes                                                                                  | No                                                                                                                                                                                                         | No                                                                                                |
| Development Effort                             | [tree-sitter](https://crates.io/crates/tree-sitter)                                  | [syntect](https://crates.io/crates/syntect) works with `.sublime-syntax` and `.tmLanguage` configuration files                                                                                             | No                                                                                                |
| Syntax Configurations                          | Active Community                                                                     | `.sublime-syntax` collections in [sublime-text packages](https://github.com/sublimehq/Packages), `.tmLanguage.json` files in [vscode extensions](https://github.com/microsoft/vscode/tree/main/extensions) | `.vim` collections in [vim runtime/syntax](https://github.com/vim/vim/tree/master/runtime/syntax) |
| Loading Syntax Configurations                  | Dynamic library (`.so`, `.dll`) loading for C-parsers, file reading for WASM-parsers | File reading                                                                                                                                                                                               | File reading                                                                                      |

Once editors parsed the tokens from a source code text file, it needs another config to give these tokens different colors to make it colorful. Here comes the theme config (vim calls it colorscheme), and editors usually allow users to customize their themes.

- For vim, it embeds some default colorschemes in its [runtime/colors](https://github.com/vim/vim/tree/master/runtime/colors) folder.
- For vscode, it embeds some default themes in its [extensions](https://github.com/microsoft/vscode/tree/main/extensions) folder, sub-folders with `theme-` prefix.
- For helix, it embeds TreeSitter queries in its [runtime/queries](https://github.com/helix-editor/helix/tree/master/runtime/queries) folder (each language also needs a `query` file to define how editors can query the tokens from TreeSitter), themes in its [runtime/themes](https://github.com/helix-editor/helix/tree/master/runtime/themes).

A theme config file actually maps each token to its color (RGB, css name, terminal ansi code) and visual effects (underline, bold, italic).

Vim/Neovim editors also have other colors, not only syntax colors for source code text files. For example Neovim has multiple highlighting groups for **floating-window**: [NormalFloat](https://neovim.io/doc/user/syntax.html#hl-NormalFloat), [FloatBorder](https://neovim.io/doc/user/syntax.html#hl-FloatBorder), [FloatTitle](https://neovim.io/doc/user/syntax.html#hl-FloatTitle), etc.

Such kind of colors are not related to programming language syntaxes, but more related for UI widgets.

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
4. Both vim's syntax engine and TextMate are regex-based, and they have a error-tolerant result when parsing partial of the source code text file, which helps a lot for real world typing. When users are typing, in most of the time, the text content is actually error-compiled, because users still not finish their typing yet. So in real world editing, most of the time, the text content is wrong from the language parser's perspective. In such cases, regex-based engines can still recognize some keywords/constants, and provide a downgraded visual effect.

Cons:

1. The TextMate community seems slowly dying. Because TextMate itself is first created by the [textmate](https://github.com/textmate/textmate) editor (a open source project from Apple). Today textmate is used by sublime-text and vscode. Sublime-text is close source software so we don't know how it implements the textmate engine. Vscode implements the textmate engine in [vscode-textmate](https://github.com/microsoft/vscode-textmate), which is written in javascript. It seems when an editor wants to use textmate engine, it will have to maintain its own textmate implementations and syntax specifications.
2. If rsvim directly uses `syntect` library, it means rsvim will have to use the `.sublime-syntax` config files, or the `.tmLanguage` config files. The biggest `.sublime-syntax` configs are maintained by sublime-text's open source packages: [sublimehq/Packages](https://github.com/sublimehq/Packages), and the biggest `.tmLanguage` configs are maintained by github linguist grammars [github-linguist/linguist/vendor/grammars](https://github.com/github-linguist/linguist/tree/main/vendor/grammars) (most these grammars are not updated for many years).
3. If rsvim directly uses `vscode-textmate` library, it means rsvim will have to implement a javascript-runtime to run the library. And also rsvim will have to use the `.tmLanguage.json` configs. The javascript engine performance can be a big issue (syntax engine should be directly embedded inside editor and be very performant). And for the `.tmLanguage.json` configs, they are maintained in [vscode/extensions](https://github.com/microsoft/vscode/tree/main/extensions).
4. Both `.sublime-syntax` and `.tmLanguage.json` grammars are not easy to extend for more rsvim colors. Because for rsvim editor, we will introduce more colors for other cases, such as floating window titles/borders, etc.

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
