# Layout

> Written by @linrongbin16, first created at 2025-11-04.

This RFC describes the layout system of TUI.

## Requirements & Use Cases

Before introducing any technical solutions, let's go through the "requirements" for Rsvim's TUI layout system.

First let's discuss "Window", "command-line" and some global widgets, they are the most important UI widgets in Rsvim as well as Vim/Neovim. They are not just simple rectangle box that shows some text contents, they contain many small UI components that improve the editing experience and provide information that help users workflow.

### Window

A window can have several sub-components:

1. Text content container: The most important component that preview the buffer content, hold a cursor inside and let user editing the buffer.
2. Line number: A vertical column component in the left side of the window, usually shows line numbers, diff symbols, diagnostic symbols, etc.
3. Window bar: A horizontal row component at the top of the window, community plugins usually use them to show a IDE-like breadcrumbs, for example [dropbar.nvim](https://github.com/Bekaboo/dropbar.nvim) for Neovim.
4. Window scoped status line: A horizontal row component at the bottom of the window, i.e. the statusline of a window.
