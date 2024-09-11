# Config File

> Written by @linrongbin16, first created at 2024-09-11.

This RFC describes the config file for RSVIM editor.

## Entry

The `.rsvim.js` or `.rsvim.ts` file is the config file where user put all their configs, and the editor trying to load on start up.

## Operations (OPs)

All the operations provided by editor are placed under the `vim` namespace, divided into several groups:

- `vim.var`: Global/local/buffer-level/window-level variables.
- `vim.ops`: Global/local/buffer-level/window-level options.
- `vim.buf`: Buffer related APIs.
- `vim.win`: Window (UI) related APIs. Note: All TUI APIs are placed under this group, by the naming style still follows the Vim's tradition, i.e. still called "window".
- And a lot more.

They are built with below principles:

- All operations are pure JavaScript functions, no variables or objects are exposed under the `vim` namespace.
- Most resources inside the editor such as buffers and windows are referred as positive integers, unless they are not suitable.
- Operating system resources follow deno's design and implementation.
- Slow operations provide both sync and async mode.

## Not Supported

Compare with other js runtimes such as [node.js](https://nodejs.org/), [deno](https://deno.com/), RSVIM editor doesn't provide:

- `document`
