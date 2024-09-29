# Builtin APIs

> Written by @linrongbin16, first created at 2024-09-27, last updated at 2024-09-29.

This RFC describe what built-in APIs that editor's js runtime support.

## Global object and functions

Js runtime provides the `Rsvim` global object just like what node/deno do, also similar to the `vim` namespace used in lua of neovim. It is almost the only way for RSVIM editor to interact with user besides TUI. This "standard library" can be divided into 2 groups:

1. Editor related APIs: Options, buffers, windows, cursor, etc.
2. General purposed APIs: IO, network, file system, IPC/RPC, child process, etc. Which is similar to most popular programming languages' standard library.

## Editor Related

For editor related APIs, we would refer to current Vim and Neovim functions and designs.

## General Purposes

For general purposed APIs/library, we would follow:

1. WinterCG's (<https://wintercg.org/>) ["Minimum Common Web Platform API"](https://common-min-api.proposal.wintercg.org/).

   > Note: There are still over 30+ class APIs in the [Common API Index](https://common-min-api.proposal.wintercg.org/#api-index) specification, which is still too much effort. We will only implement methods/properties in `globalThis` for now.

2. Deno and deno_core extensions:

   - `deno_core` extensions:
     1. [00_infra.js](https://github.com/denoland/deno_core/blob/main/core/00_infra.js)
     2. [00_primordials.js](https://github.com/denoland/deno_core/blob/main/core/00_primordials.js)
     3. [01_core.js](https://github.com/denoland/deno_core/blob/main/core/01_core.js)
   - [`deno` extensions](https://github.com/denoland/deno/tree/main/ext)

We may not provide the full standard library like deno for development effort reason, but deno's standard library will be the _**GUIDE**_ when we are looking for solutions.
