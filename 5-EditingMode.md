# Editing Mode

> Written by @linrongbin16, first created at 2025-06-13.

This RFC describes below functinalities:

1. An overview on the features of Vim/Neovim/Rsvim editor, taking scripts/commands/plugins into consideration.
2. Some high-level potential solutions and comparison among them, and finally Rsvim's choice.

## Overview

The editing mode of Vim/Neovim/Rsvim editor is one of its core features differentiates itself from other editors ([emacs](https://www.gnu.org/software/emacs/), [vscode](https://code.visualstudio.com/), [sublime-text](https://www.sublimetext.com/)). Here is the reference about Vim mode: <https://vimhelp.org/intro.txt.html#vim-modes>.

The Vim modes can be split into 2 types:

1. Basic modes.
   - Normal mode.
   - Insert mode.
   - Visual/select mode (Note: We don't distinguish between these two modes in this design).
   - Command-line (Cmdline) mode.
   - Terminal mode.
2. Additional modes.
   - Operator-pending mode.
   - Replace mode.
   - Terminal-Normal mode.
   - Insert-Normal mode.
   - Insert-Replace mode.
   - Insert-Visual/Select mode.
   - Visual-Replace mode. Note: We don't discuss this mode in this section.

The editing mode is a global state in Vim editor, the editor has and only has exactly one editing mode at a specific time. Each editing mode can be switched on different key pressing, it actually conforms to the [Finite State Machine](https://en.wikipedia.org/wiki/Finite-state_machine) design pattern.

![1](images/5-EditingMode.1.drawio.svg)

The very basic state transition is between: Normal, Insert, Visual, Operator-pending, Replace. This is the key function that improve the text editing efficiency. There are also two groups of special states for two specific scenarios:

1. Integrated Terminal: User can launch a temporary terminal inside the text editor, and let user run some shell commands without navigating outside of the editor. This feature is widely implemented by other editors such as [VsCode's integrated terminal](https://code.visualstudio.com/docs/terminal/basics), [Zed's integrated terminal](https://zed.dev/features#terminal). For Vim editor, it implements a special [Terminal Buffer](https://vimhelp.org/windows.txt.html#special-buffers) for this feature, and multiple terminal related editing modes:
   - Terminal mode: It is actually the _insert_ mode for the terminal buffer, which simulates the user input behavior in a real terminal app.
   - Terminal-Normal mode: It is actually the _normal_ mode for the terminal buffer, which allows user uses normal operators in terminal buffer.
2. Temporary modes from insert mode: Insert mode can temporarily go back to normal/visual/select/replace mode and run some operations, then automatically go back to insert mode. This feature helps further improves the editing efficiency:
   - Insert-Normal mode: It is actually the same with _normal_ mode, but it will automatically go back to insert mode after run an operator.
   - Insert-Visual/Select mode: It is actually the same with _visual/select_ mode, but it will automatically go back to insert mode after run an operator.
   - Insert-Replace mode: It is actually the same with _replace_ mode, but it will automatically go back to insert mode after run an operator.
