# TUI

> Written by @linrongbin16, 2024-08-23

This RFC describes the rendering system inside TUI.

## Hardware

Here's part lists of hardwares and benchmarks for some very popular PC/laptop computers:

| Year       |         | DIY Windows PC                                                                                      | MacBook Pro                                                                                                            |
| ---------- | ------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Since 2022 | CPU     | Intel I5-12400<br/>6 Cores (6 P-Cores), Base Frequency 2.5GHz, Max Turbo Frequency 4.40GHz          | Apple M2 Silicon Chip<br/>8 Cores (4 P-Cores, 4 E-Cores), P-Cores Base Frequency 3.5GHz, E-Cores Base Frequency 2.8GHz |
|            | Memory  | Kingston 8GB DDR4<br/>2666MT/s                                                                      | 16GB LPDDR5<br/>6400MHz                                                                                                |
|            | Storage | Samsung 512GB PCIe4.0 SSD<br/>Sequential Read/Write 4000-5000 MB/s, Random Read/Write 500-900K IOPS | 512GB SSD<br/>3200MHz                                                                                                  |
| Since 2014 | CPU     | Intel I5-4690K<br/>4 Cores, Base Frequency 3.5GHz, Max Turbo Frequency 3.90GHz                      | Intel I5 Chip<br/>Dual Cores, Base Frequency 2.6GHz, Turbo Boost Frequency 3.1GHz                                      |
|            | Memory  | Kingston 8GB DDR4<br/>1666MT/s                                                                      | 8GB DDR3L<br/>1600MHz                                                                                                  |
|            | Storage | Western Digital 512GB HDD<br/>Sequential Read/Write 150-200 MB/s, Random Read/Write 1-5 MB/s        | 512GB Flash Storage<br/>2.8GHz                                                                                         |

A very basic concept for the hardware speed from highest to lowest is: CPU > Memory > Storage. This is also the basic rules when rendering the terminal, for the same amount of data:

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
|  +------------------+ +-----------+ +--------------+ +----------+  |
|  |  Buffer Content  | |  Tabline  | |  Statusline  | |  Cursor  |  |
|  +------------------+ +-----------+ +--------------+ +----------+  |
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

<!-- https://asciiflow.com/#/share/eJztWDtvwjAQ%2FiuWZ6aqDDCWqmq3ilKxZHEbgyJRR3ISHkJIiLlDhqjKj2DsyK%2Fhl%2FTCoyLg%2BEFCWqFYJ%2Bycz5%2FvZfvEFDPyQXETtymzKXdYH9fwgEwoB97UwkPKPcdlFm7e1Cw8hr5Rv4XRJOE06jDy6diHDwuvo891NL8iCi2LraMFQq9PqOvYfep7qMMpReYNYPZgCjmJD8NCNgCJtuv6u2EBgNGBisu0xnEBG%2FxqrRTRh%2Fta5YfLne7icOpsDHNdh9nuCLVc5hOHUZ7tIx3ASKroUmWJOMwlBPjcGJxa9J3DfGFQheoK8u6EJ2McgOZIv1Bzsa6c2YLwyAz4uQt6PUjhJJkp291Mm4kOeRtAdh9wXnziB94RsxVwz90fgRS4PLElFGsu1pUzWxCbnR5l24Od7Q4jMjoweSgGo7SM13CRHpT64dhCXW8t1CJsSDwNL2Q78RKZvUgzS%2FO%2B8BHPUAo%2BnzkdOm7goQcOZbdwpRng5cxdqc1VqbW9mnlyp5%2FYmxYzA1W1IiErPf9WT%2BGTtdpMJWPtftPFBltn2C43SguwnFd4flyXX%2F7ezQtWlSeZrSpPpBT%2B88zOBnsk3B4RTtE9VAbv0v%2BWqgNcMsUWnuHZD9n8qZs%3D) -->

UI widgets tree provides the friendly interfaces for users, draws on the logical frame in the canvas in every loop, thus canvas knows the which parts of the terminal are changed and flushing these parts to hardware device. The hardware is a MxN grapheme based double-array (M is columns/width, N is rows/height), for example 240x70 on my personal laptop with a 4K Dell external monitor, thus the problem scale is `O(M * N)`.

On every loop, canvas compares current frame (after widgets drawing) and previous frame to find out the changes, then finally flushing the changes to hardware. After flushing, canvas clones and saves current frame as previous frame, for the next loop. The complexity of IO operations can vary in different scenarios:

- When user moves cursor, canvas only needs to print `O(1)` characters.
- When user edits a line, canvas needs to print `O(M)` characters.
- When user inserts/deletes a line in the middle of VIM's window, canvas needs to print `O(M * N / 2)` characters, i.e. half of the terminal.
- When user first open a file, canvas needs to print `O(M * N)` characters.

Each time a UI widget draws on canvas, the widget marks that drawing area (rectangle) as **dirty**. On every loop canvas flushes, it iterates over the current `O(M * N)` sized frame with dirty marks. The complexity of CPU and memory processing is at most `O(M * N)`.

## Curses

[Curses](<https://en.wikipedia.org/wiki/Curses_(programming_library)>) plays the role of hardware driver between RSVIM and terminal (specifically RSVIM uses [crossterm](https://github.com/crossterm-rs/crossterm)). Besides very common ASCII characters 0-9, A-Z, punctuations, etc, terminal protocols also provide extra text effects such as color, underline, bold, italic, even overlay and blur. For example:

```bash
\x1b[1;31m  # Set style to bold, red foreground.
```

Terminal protocols specify that extra escaping characters need to be prepend and append to mark these effects around the text contents. This facts add more IO operations and affect the way that how canvas flushing.

For example now we want to render below javascript sample code:

```javascript
function hello() {
  console.log("Hello, Javascript!");
}
```

Keyword `function` and literal string `"Hello, Javascript!"` are required to mark as special colors. Maybe function name `hello`, `log`, global variable `console` and even punctuations (semicolon `;`, parentheses `()` `{}`) can be mark as special colors as well.

The minimal way to print these text contents is: prepending and appending the escaping characters only once for the effected contents. For example print the `function` keyword with red color, and reset color after it:

```bash
\x1b[38;5;{31}mfunction\x1b[0m
```

The above example can split into 3 parts:

- `\x1b[38;5;{31}m`: Set red foreground color.
- `function`: The text contents.
- `\x1b[0m`: Reset all effects.

This rule requires the canvas merge continuous text contents that share same effects in one IO operation.
