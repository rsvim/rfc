# Config Entry

> Written by @linrongbin16, first created at 2024-09-11, last updated at 2025-07-14.

This RFC describes the config entry for Rsvim editor.

## Config Home and Entry File

Rsvim will use a local directory as its config home, there are 2 choices:

1. `$XDG_CONFIG_HOME/rsvim`
2. `$HOME/.rsvim`

The 1st directory has higher priority than the 2nd.

In the config home, Rsvim will use `rsvim.js` or `rsvim.ts` script as its config entry. It is similar to Vim's `.vimrc`, or Neovim's `init.lua`. Specifically, Rsvim also uses the `~/.rsvim.js` or `~/.rsvim.ts` as the config entry (`~/.rsvim` as its config home), this is an old Vim-style config entry. Thus, the config entry has 3 choices:

1. `$XDG_CONFIG_HOME/rsvim/rsvim.{js, ts}` (with `$XDG_CONFIG_HOME/rsvim` as config home).
2. `$HOME/.rsvim/rsvim.{js, ts}` (with `$HOME/.rsvim` as config home).
3. `$HOME/.rsvim.{js, ts}` (with `$HOME/.rsvim` as config home). NOTE: This is an old Vim-style config entry for a compatible user experience.
