# TUI

> Written by @linrongbin16, 2024-08-23

This RFC describes the rendering system inside TUI.

## Hardware

Here's part lists of hardwares and benchmarks for some very popular PC/laptop computers:

| Year       |         | DIY Windows PC                                                                                      | MacBook Pro                                                                             |
| ---------- | ------- | --------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Since 2022 | CPU     | Intel I5-12400<br/>6 P-Cores, Base Frequency 2.5GHz                                                 | Apple M2 Chip<br/>4 P-Cores, Base Frequency 3.5GHz<br/>4 E-Cores, Base Frequency 2.8GHz |
|            | Memory  | Kingston 8GB DDR4<br/>2666MT/s                                                                      | 16GB LPDDR5<br/>6400MHz                                                                 |
|            | Storage | Samsung 512GB PCIe4.0 SSD<br/>Sequential Read/Write 4000-5000 MB/s, Random Read/Write 500-900K IOPS | 512GB SSD<br/>3200MHz                                                                   |
| Since 2014 | CPU     | Intel I5-4690K<br/>4 Cores, Base Frequency 3.5GHz                                                   | Intel I5 Chip<br/>Dual Cores, Base Frequency 2.6GHz                                     |
|            | Memory  | Kingston 8GB DDR4<br/>1666MT/s                                                                      | 8GB DDR3L<br/>1600MHz                                                                   |
|            | Storage | Western Digital 512GB HDD<br/>Sequential Read/Write 150-200 MB/s, Random Read/Write 1-5 MB/s        | 512GB Flash Storage<br/>Read/Write 700-1000 MB/s                                        |

A very basic concept for the hardware speed from highest to lowest is: CPU > Memory > IO Devices. This is also the basic rules when rendering the terminal, for the same amount of data:

- CPU calculating/processing is preferred over flushing to IO devices.
- Sequential flushing to IO devices is preferred over random flushing.

## Architecture

The rendering system splits into 3 layers: UI widgets tree, canvas and hardware device. It looks like:

```text
+--------------------------------------------------------------------+
|  UI Widgets Tree                                                   |
|                               +--------+                           |
|                               |  Root  |                           |
|                               +---+----+                           |
|                                   |                                |
|                                   v                                |
|                         +--------------------+                     |
|                         |  Window Container  |                     |
|                         +---------+----------+                     |
|                                   |                                |
|           +-----------------+-----+--------+--------------+        |
|           v                 v              v              v        |
|  +-----------------+  +-----------+ +--------------+ +----------+  |
|  |  Text Contents  |  |  Tabline  | |  Statusline  | |  Cursor  |  |
|  +-----------------+  +-----------+ +--------------+ +----------+  |
|                                                                    |
+---------------------------------+----------------------------------+
                                  |
                                  v
+--------------------------------------------------------------------+
|  Canvas                                                            |
|                                                                    |
|            +----------------------------------+                    |
|            |  Previous Frame                  |                    |
|            |      +---------------------------+------+             |
|            |      |  Current Frame            |      |             |
|            |      |                                  |             |
|            |      |                           |      |             |
|            |      |                                  |             |
|            |      |                           |      |             |
|            |      |                                  |             |
|            |      |                           |      |             |
|            +------+   --  --  --  --  --  -- -+      |             |
|                   |                                  |             |
|                   +----------------------------------+             |
|                                                                    |
|                                                                    |
+---------------------------------+----------------------------------+
                                  |
                                  v
+--------------------------------------------------------------------+
|                                                                    |
|                                                                    |
|                         Hardware Device                            |
|                                                                    |
|                                                                    |
+--------------------------------------------------------------------+
```

UI widgets tree provides high-level friendly interfaces to interact with other editor logics, draws on the canvas in every loop, thus canvas knows the which parts of the terminal are changed and flushing these parts to hardware device. The hardware is a `M x N` grapheme based double-array (`M` is columns/width, `N` is rows/height), for example 240x70 on my personal laptop with a 4K Dell external monitor, thus the problem scale is `O(M * N)`.

After widgets drawing (on every loop), canvas compares current frame and previous frame to find out the changes, then finally flushes to hardware. After flushing, canvas clones and saves current frame as previous frame for the next loop. The worst complexity of IO operations is `O(M * N)`, it can vary in different scenarios:

- When user moves cursor, canvas only needs to print `O(1)` characters.
- When user edits a line, canvas needs to print `O(M)` characters.
- When user inserts/deletes a line in the middle of VIM's window, canvas needs to print `O(M * N / 2)` characters, i.e. half of the terminal.
- When user first open a file, canvas needs to print `O(M * N)` characters.

## Curses

[Curses](<https://en.wikipedia.org/wiki/Curses_(programming_library)>) plays the role of hardware driver between RSVIM and terminal (specifically RSVIM uses [crossterm](https://github.com/crossterm-rs/crossterm)). Besides very common ASCII characters 0-9, A-Z, punctuations, etc, [escaping codes](https://en.wikipedia.org/wiki/ANSI_escape_code) also provide extra text effects such as color, underline, bold, italic, even overlay and blur. For example:

```bash
\x1b[1;31m  # Set style to bold, red foreground.
```

For example now we want to render below javascript sample code:

```javascript
function hello() {
  console.log("Hello, Javascript!");
}
```

Text editor needs to mark different colors for the words:

- Keyword `function`.
- Literal string `"Hello, Javascript!"`.
- Function name `hello`, `log`.
- Global variable `console`.
- And even punctuations: semicolon `;`, parentheses `()`, brackets `{}`.

Escaping codes need to prepend and append extra codes to add these effects, this also increase the payload flushing to terminal device. For example now we want to print the `function` keyword with <b style='color:red'>red</b> color, and reset color after it:

```bash
\x1b[38;5;{31}mfunction\x1b[0m
```

The above example can split into 3 parts:

- `\x1b[38;5;{31}m`: Set red foreground color.
- `function`: The text contents.
- `\x1b[0m`: Reset all effects.

This requires the canvas merge continuous text contents that share same effects in one IO operation.
