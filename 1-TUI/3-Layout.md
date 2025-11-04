# Layout

> Written by @linrongbin16, first created at 2025-11-04.

This RFC describes the layout system of TUI.

## Requirements & Use Cases

Before introducing any technical solutions, let's go through the "requirements" for Rsvim's TUI layout system.

### "Window" and "Command-line" Widget

"Window" and "command-line" are the most important UI widgets in Rsvim (also Vim/Neovim). Window displays the text content of a file, or just part of the text content if the file is quite large.
