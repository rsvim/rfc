# 1. TUI

This RFC describes the basic TUI system architecture.

## Hardware Device

RSVIM uses [crossterm](https://crates.io/crates/crossterm) library as the hardware driver to handle user keyboard/mouse inputs, and render the text-based contents on the terminal such as [Gnome Terminal](https://en.wikipedia.org/wiki/GNOME_Terminal), [macOS iTerm2](https://iterm2.com/), [Windows Terminal](https://aka.ms/terminal), [kitty](https://sw.kovidgoyal.net/kitty/), [WezTerm](https://wezfurlong.org/wezterm/index.html), [Alacritty](https://alacritty.org/), etc.

Here's a very simple hardware-level event loop for RSVIM:

```text
 -------------------
|   keyboard/mouse  |
|       input       |
 -------------------
         ↓
 -------------------
|      editing      |
|        or         |    ----------
| type `:q` to quit | → |   exit   |
 -------------------     ----------
         ↓
 -------------------
|  terminal output  |
 -------------------
```

## UI Framework

The terminology _**UI**_ here is more close to the concept of most GUI and even Web/CSS frameworks, for example [Qt 6](https://doc.qt.io/qt-6/index.html), [tkinker](https://docs.python.org/3/library/tkinter.html#module-tkinter), and [Material UI](https://mui.com/material-ui/).

People would ask: (Neo)VIM is just a simple terminal app that editing a file, why do we talk about GUI framework here? - Yes and No.

(Neo)VIM today is definitely not just some simple window/buffer layout, it's extending to more complicated UI components. By introducing some GUI framework's designs and concepts, it helps to build better TUI application. While on the other hand, (Neo)VIM TUI is indeed much simpler than **Qt** and **Material UI**, we don't have to copy all of them but just part of.

In this section, we will introduce a stack-based UI components tree, which is a classic design been widely used by many GUI frameworks/libraries. For example we have a terminal application below:

```text
     --------------
     |            |
--------------------------------
| A  |    B       |            |
|    |            |            |
|    --------------            |
|                              |
|  --------------------        |
|  |                  |        |
|  | C                |        |
|  |     ----------------      |
|  |     |     ------   |      |
|  |     |  D  | E  |   |      |
|  |     |     ------   |      |
|  ------|              |      |
|        ----------------      |
--------------------------------
```

`A` is the root component of the tree, we would treat it as the terminal where the application is running on. It has 3 direct child components `B`, `C` and `D`:

- The upper part of `B` is outside of `A`, we can only see the lower part of it.
- The lower-right part of `C` is covered by `D`.
- `D` also has a child component `E` inside of it. Note: `E` is actually also inside of `C` and `A`.

Once there's user keyboard/mouse events, the TUI framework calculates the event position, tells us where does it happen:

- Is mouse clicking on `A` or `E`?
- Is keyboard editing something in `C` or `D`?

And try to dispatch the event to corresponding UI component. Each component can bind a user handler on them to handle the events happened on them. If a component doesn't have any handler, then the event will be dispatched to its parent component, and goes on with its ancestor component if also ignored by the parent.

For example now we have a mouse clicked on the center of component `E`, it will be dispatched in order: `E` => `D` => `C` => `A`.

On this basis, the (Neo)VIM UI components such as **window**, **statusline**, **column**, **float window** are all specialized components.

## Rendering

We discuss the terminal rendering in this section. After processing the user keyboard/mouse inputs and updating the UI tree, it usually:

- Changes the text content of the UI.
- Changes the UI components layout, size, etc.

Finally these logical changes will be merged into the text changes in plane 2D coordinate system, and flushed to the terminal device.
