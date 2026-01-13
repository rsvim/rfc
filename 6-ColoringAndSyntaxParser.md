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

> NOTE: We don't count [language-server-protocol](https://microsoft.github.io/language-server-protocol/) as syntax parser, it can be a big improvement but not mandatory.

### Ecosystem

Two more components are important as well for a coloring system:

- Syntax rule for each different programming language. A language has its own rule/parser to define its own syntax, the definition is loaded by the syntax parser to tokenize the text files of that language, generates the structured/tokenized information for the text files that opened by an editor.
- Theme configure for each different themes (Vim/Neovim call it [colorscheme](https://vimhelp.org/syntax.txt.html#%3Acolorscheme)). A theme defines how it actually looks for each token of a text file, e.g. color (frontground, background) and visual effects (bold, italic, underline, etc) of the text.

These two components can have hundreds/thousands or more configurations, because a programming language can have a rule configuration, a theme/colorscheme can have a theme configuration. They are mostly maintained by the open source community due to the development effort, i.e. users participate in the development and testing of all these syntax rules and color themes.

### Extra Colors

Do remember syntax highlighting doesn't provide all colors in the editor, we may want to provide extra colors and visual effects for UI components, such as:

- UI components: background, cursor, gutter, floating-window, status-line, win-bar, title-bar, etc.
- Diagnostics
- Message print level: error, warning, info, hint, etc.

The colors of these UI components are not syntax highlighting, but we want to include all these parts into the theme configuration, i.e. Rsvim's theme configuration will contain configurations for both syntax highlighting and UI components.

## Solution

### Comparison

Here is a comparison list for several solutions:

| No. | Category                                       | Tree-sitter                                                                                                                                                | TextMate                                                                                                                                                                                                                        | Vim Syntax Engine                                                                                                |
| --- | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| 1   | Quality                                        | Most Accurate                                                                                                                                              | Fine                                                                                                                                                                                                                            | Worst                                                                                                            |
| 2   | Performance (especially on opening a new file) | Lower than regex engine (need more investigation)                                                                                                          | Fine                                                                                                                                                                                                                            | Highest                                                                                                          |
| 3   | Incremental Parsing                            | Yes                                                                                                                                                        | No                                                                                                                                                                                                                              | No                                                                                                               |
| 4   | Development Effort                             | [tree-sitter](https://crates.io/crates/tree-sitter)                                                                                                        | [syntect](https://crates.io/crates/syntect) works with `.sublime-syntax` and `.tmLanguage` configuration files                                                                                                                  | No                                                                                                               |
| 5   | Syntax Configurations                          | Actively maintained by community                                                                                                                           | `.sublime-syntax` collections in [sublime-text packages](https://github.com/sublimehq/Packages), `.tmLanguage` collections in [github linguist grammars](https://github.com/github-linguist/linguist/tree/main/vendor/grammars) | `.vim` collections in [vim runtime/syntax](https://github.com/vim/vim/tree/master/runtime/syntax)                |
| 5   | Theme Configuration                            | Implement from scratch                                                                                                                                     | Need extensions for extra UI components                                                                                                                                                                                         | Implement from scratch, cannot leverage neither existing Vim `.vim` colorschemes, nor Neovim `.lua` colorschemes |
| 6   | Out of Box                                     | C/C++ compiler and `tree-sitter` cli are needed load C parser dynamic library (`so`, `dll`), no extra tools needed for WASM parsers (but rarely supported) | No                                                                                                                                                                                                                              | No                                                                                                               |

And here is the pros & cons for each solution:

|                   | Pros                                             | Cons                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tree-sitter       | - Best tokenization quality (most important)<br> | - C compiler and `tree-sitter` are needed (have to provide extra solutions to avoid this)                                                                                                                                                                                                                                                                                                             |
| TextMate          | - Good performance                               | - Syntax configurations are not actively maintained<br>- Need to integrate with `.sublime-syntax` structure for UI components<br>- `.sublime-syntax` files can have a **branding** issue (feel bad to use another editor's resource)<br>- Low-quality results should work fine for coloring system, however as more features depend on the parser results, it can lead to worse and buggy experiences |
| Vim Syntax Engine | - Good performance                               | - Need to implement from scratch<br>- Cannot leverage neither existing Vim `.vim` colorschemes, nor Neovim `.lua` colorschemes<br>- Same impact with TextMate on low-quality results                                                                                                                                                                                                                  |

I believe tree-sitter is the best choice because both itself and its community are actively maintained, it also has a clear documentation. The challenges for Rsvim are:

1. How to avoid the slow parsing speed on super big source files on first opening.
2. How to make the syntaxes pluggable. As we want to allow user install/upgrade/remove a language parser independently, we may not want to directly embed tree-sitter grammars into the editor, but distribute them as a independent plugin.
3. How to automatically download pre-built C parser dynamic libraries, to avoid local compiling parsers for users.

### Parsing

There are mostly two cases, i.e. total parsing and incremental parsing:

- Total parsing on opening a buffer
  1. Buffer stage:
     1. (Current) Read the whole source file into memory, and create new buffer with it.
     2. (Add) Find the corresponding tree-sitter language based on file extension.
     3. (Add) Create a new tree-sitter parser with corresponding language.
     4. (Add) Parse the buffer into tokens with the parser.
  2. Render stage:
     1. (Add) Query the tree-sitter highlight associated with the tree-sitter parser.
     2. (Add) Rendering every cells inside the Window with the highlight.
- Incremental parsing on editing a buffer
  1. Buffer stage:
     1. (Current) Handle editing logic and change buffer content.
     2. (Add) Parse the changed buffer.
  2. Render stage: same with the "Total parsing" case.

The most time-costing step should be the 4th step in total parsing when opening a buffer. The problem scale grows with the buffer size. A decision need to make, that whether we should parsing the whole buffer in async, to avoid blocking main thread of the editor.

There already have some existing work:

- [Neovim's treesitter async parsing improvement](https://github.com/neovim/neovim/pull/31631).
- Different languages and grammar implementations can have huge performance differences. Since they are maintained by different communities, which we cannot rely on.
