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

For general purposed APIs/library, we would follow below principles:

1. Compatible with [WinterCG](https://wintercg.org/) standard.
2. Exclude APIs that only works for browsers and web applications.

### WinterCG Standard

WinterCG's [Minimum Common Web Platform API](https://common-min-api.proposal.wintercg.org/) has over 30+ class APIs in the specification.

- Phase-1: Implement the `globalThis` and all the methods/properties in it.

### Deno Builtin Extensions

Deno's builtin extensions are a great set of implements, we would follow their API design and implementations. We may not provide the full standard library like deno, but deno's standard library will be the _**GUIDE**_ when we are looking for solutions.

- `deno_core` extensions:
  1.  [00_infra.js](https://github.com/denoland/deno_core/blob/main/core/00_infra.js)
  2.  [00_primordials.js](https://github.com/denoland/deno_core/blob/main/core/00_primordials.js)
  3.  [01_core.js](https://github.com/denoland/deno_core/blob/main/core/01_core.js)
- [`deno` extensions](https://github.com/denoland/deno/tree/main/ext)
  1. [fs](https://github.com/denoland/deno/tree/main/ext/fs): File system and IO.
  2. [net](https://github.com/denoland/deno/tree/main/ext/net): Network, tls and dns.
  3. [tls](https://github.com/denoland/deno/tree/main/ext/tls): TLS handling.
  4. [url](https://github.com/denoland/deno/tree/main/ext/url): `URL`.
  5. [url](https://github.com/denoland/deno/tree/main/ext/url): `URL`.
