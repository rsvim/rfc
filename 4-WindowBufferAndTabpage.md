# Window, Buffer and Tabpage

> Written by @linrongbin16, 2024-09-04

This RFC describes below functinalities:

- [windows](https://vimhelp.org/windows.txt.html)
- [buffers](https://vimhelp.org/windows.txt.html#buffers)
- [tabpage](https://vimhelp.org/tabpage.txt.html)
- [popup](https://vimhelp.org/popup.txt.html#popup) (or [floating window](https://neovim.io/doc/user/api.html#_floating-windows))

The main functionality still follows (Neo)Vim (see [References](#references)), while may involve some upgrades and break changes.

## Concepts

### Window

- [Normal window](https://vimhelp.org/windows.txt.html#windows), the viewport of a buffer.
- [Preview window](https://vimhelp.org/windows.txt.html#preview-window), a special window to show/preview a file, the window is non-editable and non-interactive.
- [Popup window](https://vimhelp.org/popup.txt.html#popup) (or [floating window](https://neovim.io/doc/user/api.html#_floating-windows) in Neovim), the window goes on top of other windows, specified by the Z-index property.

### Buffer

- [File buffer](https://vimhelp.org/windows.txt.html#buffers), in-memory text of a file.
- [Quickfix buffer](https://vimhelp.org/windows.txt.html#special-buffers), errors or locations list.
- [Help buffer](https://vimhelp.org/windows.txt.html#special-buffers), vim help.
- [Terminal buffer](https://vimhelp.org/windows.txt.html#special-buffers), terminal emulator.
- [Directory buffer](https://vimhelp.org/windows.txt.html#special-buffers), Directory/folder (filesystem).
- [Scratch buffer](https://vimhelp.org/windows.txt.html#special-buffers), Discardable text.
- [Unlisted buffer](https://vimhelp.org/windows.txt.html#special-buffers), Unlisted buffers.

### Tabpage

[Tabpage](https://vimhelp.org/tabpage.txt.html) manages a group of windows, similar to the concepts of [workspaces](https://help.gnome.org/users/gnome-help/stable/shell-workspaces.html) in Gnome Desktop, [spaces](https://support.apple.com/guide/mac-help/work-in-multiple-spaces-mh14112/14.0/mac/14.0) in macOS, [desktops](https://support.microsoft.com/en-us/windows/multiple-desktops-in-windows-36f52e38-5b4a-557b-2ff9-e1a60c976434) in Windows.

## Migrate to New Engine

We need to re-think these features and designs, to both keep most functions compatible with (Neo)Vim to avoid user complaints, and also to be able to be migrated to new TUI engine.

## References

- [Vim help files - windows.txt](https://vimhelp.org/windows.txt.html)
- [Neovim user documentation - windows](https://neovim.io/doc/user/windows.html)
