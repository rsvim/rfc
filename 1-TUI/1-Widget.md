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

The ownership guarantees:

- Children will be destroyed when their parent is.
- A node has two coordinate systems: relative and absolute. The relative coordinate is based on its parent, the absolute coordinate is based on the terminal device.
- Children are displayed inside their parent's geomtric shape, clipped by their boundaries. While the logical shape can still be infinite on the imaginary canvas.
- Children always have higher priority over their parent to display. While Z-index arranges the display priority when multiple children overlap on each other, for children under the same parent, higher Z-index has higher priority to display.
- Common attributes of children are implicitly inherited from their parent, for example `visible` and `enabled`, unless they are been explicitly been changed.

## Coordinate System

Fortunately in VIM editor, all widgets are rectangles. They have their own shapes: size (height and width), position (x/y, row/column). This is the 2-dimensional coordinate system:

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

When it comes to the terminal device, we set the `(0,0)` coordinate as the top-left corner of the terminal device:

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

Widgets can be stacking and overlapping: higher Z-index has higher priority than lower Z-index to display.

```text
(0,0)-------------------------------------(width,0)--->X
  |                                             |
  |  Terminal                                   |
  |                                             |
  |      +------------------------+             |
  |      | A: Z-index 20          |             |
  |      |   +-------------------------+        |
  |      |   | B: Z-index 50      |    |        |
  |      |   |                         |        |
  |      |   |                    |    |        |
  |      |   |                         |        |
  |      |   |                    |    |        |
  |      +---|-- -- -- -- -- -- --+    |        |
  |          |                         |        |
  |          +-------------------------+        |
  |                                             |
(0,height)--------------------------------(width,height)
  |
  |
  v
  Y
```

## Event Handling and Dispatching

With coordinate system, we can find out where does a keyboard/mouse event happen, and dispatch it from the leaf nodes to root node in the widget tree, which is totally intuitive. Each widget node has a default event handler (callback method) binding on it, the default behavior is simply doing nothing. Users can bind their own handlers to do extra logics.

For a handler/callback, it returns `true` or `false` to the TUI engine:

- When returns `true`, it tells the engine to stop dispatching to its parent node, i.e. the event is consumed by this handler.
- When returns `false`, it tells the engine to continue dispatching to its parent node, i.e. the event is not consumed by this handler even it's been processed. The default handler returns `false`.