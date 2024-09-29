# Builtin APIs

> Written by @linrongbin16, first created at 2024-09-27, last updated at 2024-09-29.

This RFC describe what built-in APIs that editor's js runtime support.

## `vim` global object

Js runtime provides the `vim` global object, so let users write scripts with:

```javascript
vim.opt.setLineWrap(true);
vim.opt.setTabSpace(2);
```

This design is similar to node, deno and neovim with lua.

## Editor Related

Editor related APIs will be following the design of Vim and Neovim:

- `vim.opt`
- `vim.buf`
- `vim.win`
- And more.

## General Purposes

General purposes library (similar to standard libraries for some general programming languages) will follow two principles:

1. Compatible with [WinterCG](https://wintercg.org/) and popular js runtimes such as node/deno.
2. POSIX/Unix/Linux like, unless it is Windows specific.

We are not going to provide a full set of library like node or deno. Because they are mostly for general purposes development. Only a small subset will be implemented, such as file system, child process, operating system, data structures, IO, network, etc. And also we may want to remain a small group to avoid too much maintain effort:

- [deno_core extensions](https://github.com/denoland/deno_core/tree/main/core)
- [deno extensions](https://github.com/denoland/deno/tree/main/ext)
