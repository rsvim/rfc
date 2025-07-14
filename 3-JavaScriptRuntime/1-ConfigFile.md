# Config File

> Written by @linrongbin16, first created at 2024-09-11, last updated at 2025-07-14.

This RFC describes the config file for Rsvim editor.

## Config Entry

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
- The design of editor APIs follow both [Neovim API](https://neovim.io/doc/user/api.html) and [Vim help](https://vimhelp.org/).
- The design of operating system APIs follow [MDN Web APIs](https://developer.mozilla.org/en-US/docs/Web/API).
- Primitive values are preferred when APIs return a value to js script, unless it's necessary.
- Slow operations provide both sync and async mode.

## Not Supported

Compare with other js runtimes such as [node.js](https://nodejs.org/), [deno](https://deno.com/), Rsvim editor doesn't provide:

- `document`
