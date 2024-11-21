# Canvas

> Written by @linrongbin16, first created at 2024-08-23, last updated at 2024-11-21.

This RFC describes the canvas system inside TUI.

## Architecture

The rendering system splits into 3 layers: UI widgets tree, canvas and hardware device. It looks like:

![1](../images/1-TUI-2-Canvas.1.drawio.svg)

UI widgets tree provides high-level friendly interfaces to interact with other editor logics, draws on the canvas in every loop, thus canvas knows the which parts of the terminal are changed and flushing these parts to hardware device. The hardware is a `M x N` grapheme based double-array (`M` is columns/width, `N` is rows/height), for example 240x70 on my personal laptop with a 4K Dell external monitor, thus the problem scale is `O(M * N)`.

After widgets drawing (on every loop), canvas compares current frame and previous frame to find out the changes, then finally flushes to hardware. After flushing, canvas clones and saves current frame as previous frame for the next loop. The worst complexity of IO operations is `O(M * N)`, it can vary in different scenarios:

- When user moves cursor, canvas only needs to print `O(1)` characters.
- When user edits a line, canvas needs to print `O(M)` characters.
- When user inserts/deletes a line in the middle of VIM's window, canvas needs to print `O(M * N / 2)` characters, i.e. half of the terminal.
- When user first open a file, canvas needs to print `O(M * N)` characters.

## Escaping Codes

### Display Attributes

[Curses](<https://en.wikipedia.org/wiki/Curses_(programming_library)>) plays the role of hardware driver between RSVIM and terminal (specifically RSVIM uses [crossterm](https://github.com/crossterm-rs/crossterm)). Besides very common ASCII characters 0-9, A-Z, punctuations, etc, [escaping codes](https://en.wikipedia.org/wiki/ANSI_escape_code) also provide extra text effects such as color, underline, bold, italic, even overlay and blur. For example we want to render below javascript sample code:

```javascript
function hello() {
  console.log("Hello, Javascript!");
}
```

We need to add colors for the words:

- Keyword `function`.
- Literal string `"Hello, Javascript!"`.
- Function name `hello`, `log`.
- Global variable `console`.
- And even punctuations: semicolon `;`, parentheses `()`, brackets `{}`.

Terminal needs to prepend and append extra escaping codes to add these effects, and increases the flushing payload. For example we want to print the `function` keyword with red color, and reset color after it:

```bash
\x1b[38;5;{31}mfunction\x1b[0m
```

The above sequence can be split into 3 parts:

- `\x1b[38;5;{31}m`: Set red foreground color.
- `function`: The text contents.
- `\x1b[0m`: Reset all effects.

Surrounding these escaping codes one by one for each character (`f`, `u`, `n`, `c`, `t`, `i`, `o`, `n`) also works, but the increased overhead is worst. Minimal overhead requires the canvas merge consequent text contents that sharing the same effects.

### Control Sequences

There are some other escaping codes to control terminal (it's called [Command](https://docs.rs/crossterm/latest/crossterm/trait.Command.html) in crossterm):

- Cursor: Move up, down, left, right, or to specific position. Show and hide, save and restore position.
- Clear terminal: All, cursor line, column cells up from cursor, column cells down from cursor.
- Set size: Set terminal buffer size.

Consider below example when we're editing a file inside VIM editor:

![2](../images/1-TUI-2-Canvas.2.drawio.svg)

There are several UI widgets:

- Tabline (A), statusline (B): Global widgets for an editor instance.
- Sidebar (C), structure outline (D): Special window that for showing extra directory structures and source code structures, not file editing.
- Window 1 (F) and 2 (H), and their winbar (E and G): Special window that for file editing, each of them has a special winbar widget for extra navigation info.

For such a high-frequency scenarios, canvas will have to do several kind of steps:

```text
1. Save cursor position.
2. Foreach changed line in whole canvas:
3.     Foreach changed contents on the line:
4.       Move cursor to the start position of the changed contents.
5.       Print the contents with required display effects.
6. Restore cursor position.
7. Do (the real) cursor movement.
```
