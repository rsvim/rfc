# Windows and Buffers

> Written by @linrongbin16, first created at 2024-09-04.

This RFC describes below functinalities:

- [windows](https://vimhelp.org/windows.txt.html)
- [buffers](https://vimhelp.org/windows.txt.html#buffers)
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

## References

- [Vim help files - windows.txt](https://vimhelp.org/windows.txt.html)
- [Neovim user documentation - windows](https://neovim.io/doc/user/windows.html)
