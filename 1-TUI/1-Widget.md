# Widget

> Written by @linrongbin16, 2024-08-28

This RFC describes the widget tree inside TUI.

## Design

The design of TUI engine is deeply influenced by [Qt](https://www.qt.io/) GUI framework:

- Each widget is a node on the widgets tree.
- Parent node owns its children nodes.
- Coordinate system (shapes and layers).
- Keyboard/mouse events dispatching and handling.

Consider below example, when we're editing a file inside VIM editor:

```text
+-------------------------------------------------------------------------+
| A:Tabline                                                               |
+---------------------+---------------------------------------------------|
| C:Window-1          | D:Window-2                                        |
|                     |                                                   |
|                     |                                                   |
| Some text           | Some text contents here...                        |
| contents here...    |                                                   |
|                     | +--------+                                        |
|                     | |E:Cursor|                                        |
|                     | +--------+                                        |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
|                     |                                                   |
+---------------------+---------------------------------------------------|
| B:Statusline (global)                                                   |
+-------------------------------------------------------------------------+
```

While the widgets tree looks like:

```text
+----------------+
| Root Container |
+-------+--------+
        |
        +---------------------+-------------------+--------------------+
        |                     |                   |                    |
        v                     v                   v                    v
+----------------+   +----------------+   +----------------+   +----------------+
| A: Tabline     |   | B: Statusline  |   | C: Window-1    |   | D: Window-2    |
+----------------+   +----------------+   +-------+--------+   +-------+--------+
                                                  |                    |
                                                  |                    +-------------------+
                                                  |                    |                   |
                                                  v                    v                   v
                                          +----------------+   +----------------+   +----------------+
                                          | Window-1 Texts |   | Window-2 Texts |   | E: Cursor      |
                                          +----------------+   +----------------+   +----------------+
```

NOTE: The `Root Container` and `Window` nodes are pure logical nodes, i.e. they only have shapes and can arrange their children nodes layout, but no text contents.

## Ownership

- Children will be destroyed when their parent is.
- Each node has two coordinate systems: relative and absolute. The relative coordinate is based on its parent, which is easier for user. The absolute coordinate is based on the terminal device, which is faced to hardware rendering.
- Children are displayed inside their parent's geomtric shape, clipped by their boundaries. While the logical shape can still be infinite on the imaginary canvas.
- Children always covers their parent. The Z-index arranges the display priority of the content stack when multiple children overlap on each other, for widgets under the same parent, higher Z-index has higher priority to display.
- Common attributes of children are implicitly inherited from their parent, for example `visible` and `enabled`, unless they are been explicitly been changed.

## Coordinate System

All widgets are rectangles (fortunately we don't need to involve too much complicated graphic calculations), and have their own shapes: size (height and width), position (x/y, row/column). This is the 2-dimensional coordinate system:

```text
                Y
                |
                |
                |
                |
              (0,1)
                |
X------(-1,0)-(0,0)-(1,0)------->
                |
              (0,-1)
                |
                |
                |
                |
                v
```

When it comes to the terminal device, we set the `(0,0)` coordinate as the top-left corner of the terminal device. Thus it becomes to:

```text
(0,0)------------------------(width,0)-->X
  |                                |
  |  Terminal                      |
  |                                |
  |                                |
  |                                |
  |                                |
  |                                |
  |                                |
  |                                |
  |                                |
(0,height)-------------------(width,height)
  |
  v
  Y
```

- Widgets can be stacking and overlapping, this involves the Z-axis (Z-index), higher Z-index covers lower Z-index.

## Event Handlers and Dispatching

- Once an event happens, the event will be dispatched from leaf nodes to root nodes.
- All widgets have a default event handler (callback methods) binding on it, the default behavior is simply doing nothing. But users can bind their own handlers to listen to the events and do extra logics.
- An event is been consumed if a node has an event handler to consume it, or it will continue to be dispatched to its parent node. There's only one exception: when the handler tells the UI engine to continue dispatching this event to its parent node, even the event is been consumed by current handler, it will still continue to be dispatched to its parent node.
