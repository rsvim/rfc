# Editing Mode

> Written by @linrongbin16, first created at 2025-06-13.

This RFC describes below functinalities:

1. An overview on the features of Vim/Neovim/Rsvim editor, taking scripts/commands/plugins into consideration.
2. Some high-level potential solutions and comparison among them, and finally Rsvim's choice.

## Overview

The editing mode of Vim/Neovim/Rsvim editor is one of its core features differentiates itself from other editors ([emacs](https://www.gnu.org/software/emacs/), [vscode](https://code.visualstudio.com/), [sublime-text](https://www.sublimetext.com/)). Here are some references about Vim and Neovim:

- Vim modes: <https://vimhelp.org/intro.txt.html#vim-modes>.
- Neovim modes: <https://neovim.io/doc/user/vimindex.html>.

To make it simple, let's ignore some unimportant modes and only keeps the most important modes. The modes can be split into 2 types:

1. Basic modes.
   - Normal mode.
   - Insert mode.
   - Visual/select mode (Note: We don't distinguish between these two modes in this design).
   - Command-line (Cmdline) mode.
   - Terminal mode.
2. Additional modes.
   - Operator-pending mode.
