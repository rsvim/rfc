# Builtin APIs

> Written by @linrongbin16, first created at 2024-09-27.

This RFC describe what operations does js runtime support.

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

General purposes APIs (similar to standard libraries for some general programming languages) will be compatible with [WinterCG](https://wintercg.org/), also to be POSIX-like, Unix-like, Linux. We have several options:

| Option                                               | Pros                  | Cons                        |
| ---------------------------------------------------- | --------------------- | --------------------------- |
| [Deno extensions](https://github.com/denoland/deno)  | High quality          | Codebase too large          |
| [Node.js APIs](https://nodejs.org/api/index.html)    | Widely used           | Chaotic, codebase too large |
| [rustix](https://github.com/bytecodealliance/rustix) | POSIX/Unix/Linux like | Not js runtimes             |

We are not going to provide a full set of APIs, because big standard libraries are mostly for general purposes developing. Only a small subset will be implemented, such as file system, child process, operating system, data structures, IO, network, etc. And also we may want to remain a small group instead of being super big because it may not worth it:

1. Part of deno extensions: for compatibility with the web world: ECMAScript standards, WinterCG and popular js runtimes.
2. Port part of rustix APIs into javascript: for file system, IO, network, child process, operating systems, etc.
