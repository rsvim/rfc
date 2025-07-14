# Config Entry

> Written by @linrongbin16, first created at 2024-09-11, last updated at 2025-07-14.

This RFC describes the config entry for Rsvim editor.

## Config Home and Entry File

Rsvim will use a local directory as its config home, there are 2 choices:

1. `$XDG_CONFIG_HOME/rsvim`
2. `$HOME/.rsvim`

The 1st directory has higher priority than the 2nd.

In the config home, Rsvim will use `rsvim.js` or `rsvim.ts` script as its config entry. It is similar to Vim's `.vimrc`, or Neovim's `init.lua`. Specifically, Rsvim also uses the `~/.rsvim.js` or `~/.rsvim.ts` as the config entry (`~/.rsvim` as its config home), this is an old Vim-style config entry. Thus, the config entry has 3 choices:

1. `$XDG_CONFIG_HOME/rsvim/rsvim.{js, ts}` (with `$XDG_CONFIG_HOME/rsvim` as config home).
2. `$HOME/.rsvim/rsvim.{js, ts}` (with `$HOME/.rsvim` as config home).
3. `$HOME/.rsvim.{js, ts}` (with `$HOME/.rsvim` as config home). NOTE: This is an old Vim-style config entry for a compatible user experience.

## API Style

There will be 3 groups of JavaScript APIs in Rsvim:

- Builtin APIs: The builtin APIs provided by V8 engine, defines by ECMA standard, please see [Standard built-in objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects).
- Web APIs: Most javascript-based runtimes implements a set of Web APIs.
- Specific runtime APIs: Different javascript-based runtimes provide their own APIs, different from web browsers and server-side runtimes, Rsvim provides APIs related to Vim windows/buffers, similar to Neovim's [`vim.api`](https://neovim.io/doc/user/api.html).

> NOTE: In JavaScript world, a lot of standard APIs are provided by **global objects**, they can be access without using any `import` or `require`.

Inside scripts, JavaScript APIs are literally the only way to interact with Rsvim editor. There is no key mappings or user commands, they are actually only editor UI backed with registered js functions.

All js APIs provided by Rsvim are placed under the `Rsvim` namespace, divided into several groups:

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
