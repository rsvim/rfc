# JavaScript APIs

> Written by @linrongbin16, first created at 2025-07-14.

# API Style

There will be 3 groups of JavaScript APIs in Rsvim:

- Builtin APIs: The builtin APIs provided by V8 engine, defines by ECMA standard, please see [Standard built-in objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects).
- Web APIs: Most javascript-based runtimes implements a set of Web APIs, please see [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API). But most web APIs are for web browsers and web applications, which are not suitable for Rsvim editor. Rsvim editor will only implement a small set of them.
- Specific runtime APIs: Different javascript-based runtimes provide their own APIs, different from web browsers and server-side runtimes, Rsvim provides APIs related to Vim windows/buffers, similar to Neovim's [`vim.api`](https://neovim.io/doc/user/api.html).

Inside scripts, JavaScript APIs are literally the only way to interact with Rsvim editor. There is no key mappings or user commands, they are actually only editor UI backed with registered js functions.

## ECMA Standard Builtin APIs

Standard builtin APIs are already implemented by V8 engine, they are **global objects** that can be access without using any `import`/`require`. For example:

- [`parseFloat()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseFloat)/[`parseInt()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt)
- [`encodeURI()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURI)/[`decodeURI()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/decodeURI)

Rsvim also enables extra V8 flags to allow the `esnext` standard such as [`Temporal`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal), please see [MDN Temporarl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal) and [tc39 Proposal Temporal](https://tc39.es/proposal-temporal/).

## Web APIs

Web APIs are also **global objects**, they are a de-facto standard that are been widely implemented in most popular browsers such as Chrome/Firefox, and server-side runtimes such as Node/Deno. Some APIs are so popular that people usually think they are builtin inside JavaScript:

- [`console`](https://developer.mozilla.org/en-US/docs/Web/API/console).
- [`setTimeout()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout)/[`clearTimeout()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/clearTimeout)
- [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)
- [`document`](https://developer.mozilla.org/en-US/docs/Web/API/Window/document) (especially for browsers)

But anyway Rsvim is not a general server-side runtime, the existence of all the APIs is for serving the editing and text processing. Thus, Rsvim will only implement part of the [WinterTC](https://min-common-api.proposal.wintertc.org/) standard.

## Specific APIs

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
