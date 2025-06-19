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

## Five Modes

The very basic state transition is between: Normal, Insert, Visual, Operator-pending, Replace. This is the core function of Vim editor to improve the text editing efficiency. In the following of this section, let's call it the **Five-Modes** product design for text editing.

## Mode Variants

There are 3 groups of mode variants for 3 specific scenarios:

1. Command-line: The command-line mode actually consists of 3 variants:
   - Ex-command variant: Press ':' to input ex-command (include user-commands).
   - Search pattern forward variant: Press '/' to search pattern forward.
   - Search pattern backward variant: Press '?' to search pattern backward.
2. Integrated terminal: User can launch a temporary terminal inside the text editor, and let user run shell commands without leaving the editor. This feature is widely implemented by other editors such as [VsCode's integrated terminal](https://code.visualstudio.com/docs/terminal/basics), [Zed's integrated terminal](https://zed.dev/features#terminal). For Vim editor, it implements a special [Terminal Buffer](https://vimhelp.org/windows.txt.html#special-buffers) for this feature, and multiple mode variants related to terminal mode:
   - Terminal mode: Same with the _insert_ mode, but for terminal buffer, which simulates the user input behavior in a real terminal app.
   - Terminal-Normal mode: Same with the _normal_ mode, but for the terminal buffer, which allows user uses normal operators in terminal buffer.
3. Temporary mode variants in insert mode: Insert mode can go to temporary normal/visual/select/replace mode variants and run some operations, then automatically go back to insert mode. This feature helps further improves the insertion efficiency:
   - Insert-Normal mode: Same with _normal_ mode, but it will automatically go back to insert mode after run an operation.
   - Insert-Visual/Select mode: Same with _visual/select_ mode, but it will automatically go back to insert mode after run an operation.
   - Insert-Replace mode: Same with _replace_ mode, but it will automatically go back to insert mode after run an operation.

For rsvim, we can have a better re-design about these two specific scenarios by extending the states transition.

### Command-line Variants

We actually have 3 command-line modes, or say, we have 3 variants in command-mode:

- Ex-command variant.
- Search pattern forward variant.
- Search pattern backward variant.

![4](images/5-EditingMode.4.drawio.svg)

### Integrated Terminal

We could directly apply/copy the **Five-Modes** for the integrated terminal, with some optimizations and adaptions for terminal input scenario:

- Terminal mode: The insert mode in terminal buffer.
- Terminal-Normal mode: The normal mode in terminal buffer.
- Terminal-Visual/Select mode: The visual/select mode in terminal buffer.
- Terminal-Replace mode: The replace mode in terminal buffer.
- Terminal-Operator-pending mode: The operator-pending mode in terminal buffer.

![2](images/5-EditingMode.2.drawio.svg)

### Temporary Insert Variant Modes

The 5 temporary insert variant modes is started from insert mode, they are also copied from the **Five-Modes**, the difference is they are temporary modes and only exist for one operation:

- Insert mode.
- Insert-Normal mode.
- Insert-Visual/Select mode.
- Insert-Replace mode.
- Insert-Operator-pending mode.

![3](images/5-EditingMode.3.drawio.svg)

> Do we really need this **Temporary Insert Variant Modes** as a product feature for rsvim?
