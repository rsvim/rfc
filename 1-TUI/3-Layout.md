# Layout

> Written by @linrongbin16, first created at 2025-11-04.

This RFC describes the layout system of TUI.

## Use Cases

Before introducing any technical solutions, let's go through the requirements, e.g. use cases, for Rsvim's TUI layout system.

### Inside Widgets

First let's discuss "Window", "command-line" and some global widgets, they are the most important UI widgets in Rsvim as well as Vim/Neovim. They are not just simple rectangle box that shows some text contents, they contain many small UI components that improve the editing experience and provide information that help users workflow.

#### Window

A window can have below components:

1. Text content: The most important component that preview the buffer content, hold a cursor inside and let user editing the buffer.
2. Line number: A vertical column component in the left side of the window, usually shows line numbers, diff symbols, diagnostic symbols, etc.
3. Window bar: A horizontal row component at the top of the window, community plugins usually use them to show a IDE-like breadcrumbs, for example [dropbar.nvim](https://github.com/Bekaboo/dropbar.nvim) for Neovim.
4. Status line (window scoped): A horizontal row component at the bottom of the window, i.e. the statusline of a window.
5. Scroll bar: A vertical column component in the right side of the window, usually shows a scrollbar to indicate the current viewport position in the whole buffer.

![1](../images/1-TUI-3-Layout.1.drawio.svg)

Don't be scared by it, a complicated window can be achieved by recursively dividing a simple rectangle into several parts either horizontally or vertically. In this example, the window can be achieved by:

1. First split a rectangle into 3 rectangles horizontally, and we got:
   - "Line number" rectangle on the left side.
   - "Text content" rectangle in the middle.
   - "Scroll bar" rectangle on the right side.
2. Then split the "text content" rectangle into 3 rectangles vertically, and we got:
   - "Window bar" rectangle at the top.
   - "Text content" rectangle in the middle.
   - "Status line" rectangle at the bottom.

#### Command-Line

A command-line can have below components:

1. For user input variant, it contains:
   - A `:` indicator that indicates current command-line accepts user input commands, and the last editing mode is "Normal". If the last editing mode is "Visual"/"Select", the indicator is `:'<,'>` that indicates cmdline also accepts the visual-selected text range.
   - The user input text content as commands and arguments.

   ![2](../images/1-TUI-3-Layout.2.drawio.svg)

2. For search forward/backward, it contains:
   - A `/` or `?` indicator that indicates current command-line accepts user input pattern for searching on current buffer.
   - The user input text content as search pattern.

   ![3](../images/1-TUI-3-Layout.3.drawio.svg)

#### Global Widgets

There are some other global UI widgets, such as statusline, winbar, etc.

### Multiple Windows

Rsvim (and Vim/Neovim) allow user splitting their window vertically and horizontally, and also creating floating window. Here are some examples:

![4](../images/1-TUI-3-Layout.4.drawio.svg)

![5](../images/1-TUI-3-Layout.5.drawio.svg)

![6](../images/1-TUI-3-Layout.6.drawio.svg)

### Summary

So after took a look at these use cases, we can summarize our requirements about layout system in Rsvim:

1. We can split a parent widget (rectangle) into `N` continuous small child rectangles either horizontally or vertically.
2. We can apply some strategy on these child rectangles to tell them what position they should be. For the strategy, we can leverage the [CSS Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox) layout, since it is quite close to our use cases.

## CSS Flexbox
