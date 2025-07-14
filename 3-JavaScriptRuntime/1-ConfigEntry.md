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
- Web APIs: Most javascript-based runtimes implements a set of Web APIs, please see [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API). But most web APIs are for web browsers and web applications, which are not suitable for Rsvim editor. Rsvim editor will only implement a small set of them.
- Specific runtime APIs: Different javascript-based runtimes provide their own APIs, different from web browsers and server-side runtimes, Rsvim provides APIs related to Vim windows/buffers, similar to Neovim's [`vim.api`](https://neovim.io/doc/user/api.html).

Inside scripts, JavaScript APIs are literally the only way to interact with Rsvim editor. There is no key mappings or user commands, they are actually only editor UI backed with registered js functions.

### ECMA Standard Builtin APIs

Standard builtin APIs are already implemented by V8 engine, they are **global objects** that can be access without using any `import`/`require`. For example:

- [`parseFloat()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseFloat)/[`parseInt()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt)
- [`encodeURI()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURI)/[`decodeURI()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/decodeURI)

Rsvim also enables extra V8 flags to allow the `esnext` standard such as [`Temporal`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal), please see [MDN Temporarl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal) and [tc39 Proposal Temporal](https://tc39.es/proposal-temporal/).

### Web APIs

Web APIs are also **global objects**, they are a de-facto standard that are been widely implemented in most popular browsers such as Chrome/Firefox, and server-side runtimes such as Node/Deno. Some APIs are so popular that people usually think they are builtin inside JavaScript:

- [`console`](https://developer.mozilla.org/en-US/docs/Web/API/console).
- [`setTimeout()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout)/[`clearTimeout()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/clearTimeout)
- [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [`document`](https://developer.mozilla.org/en-US/docs/Web/API/Window/document) (especially for browsers)

But anyway Rsvim is not a general server-side runtime, the existence of all the APIs is for serving the editing and text processing. Thus, Rsvim will only implement part of the [WinterTC](https://min-common-api.proposal.wintertc.org/) standard.

### Specific APIs

For all the other specific APIs, they are placed under the `Rsvim` namespace, divided into two kinds, several groups. Each group is either **Editor APIs** or **General APIs**.

For editor APIs:

- `Rsvim.opts`: Global and global-local options.
- `Rsvim.buf`: Buffer related APIs.
- `Rsvim.win`: Window related APIs.
- And a lot more.

For general APIs:

- `Rsvim.process`: The editor process related APIs.
- And a lot more.

The editor APIs will follow design of [Neovim Lua API](https://neovim.io/doc/user/api.html) and [Vim functions](https://vimhelp.org/), the general APIs will follow [Node API](https://nodejs.org/api/n-api.html) and [Deno API](https://docs.deno.com/api/deno/~/Deno).
