# Window, Buffer and Tabpage

> Written by @linrongbin16, 2024-09-04

This RFC describes below functinalities:

- [windows](https://vimhelp.org/windows.txt.html)
- [buffers](https://vimhelp.org/windows.txt.html#buffers)
- [tabpage](https://vimhelp.org/tabpage.txt.html)
- [popup](https://vimhelp.org/popup.txt.html#popup) (or [floating window](https://neovim.io/doc/user/api.html#_floating-windows))

The main functionality still follows (Neo)Vim (see [References](#references)), while may involve some upgrades and break changes.

## Concepts

(Neo)Vim contains below windows:

- [Normal window](https://vimhelp.org/windows.txt.html#windows), the viewport of a buffer.
- [Preview window](https://vimhelp.org/windows.txt.html#preview-window), a special window to show/preview a file.

## References

- [Vim help files - windows.txt](https://vimhelp.org/windows.txt.html)
- [Neovim user documentation - windows](https://neovim.io/doc/user/windows.html)
