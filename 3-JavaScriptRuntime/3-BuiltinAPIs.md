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

General purposes APIs (similar to standard libraries for some general programming languages) will be following the POSIX-like, Unix-like, Linux. We have several options:

| Option                                                                 | Pros         | Cons               |
| ---------------------------------------------------------------------- | ------------ | ------------------ |
| [Deno standard library](https://jsr.io/@std)                           | High quality | Codebase too large |
| [Node.js APIs](https://nodejs.org/api/index.html)                      |              |                    |
| [LLRT modules](https://github.com/awslabs/llrt/tree/main/llrt_modules) |              |                    |
| [rustix](https://github.com/bytecodealliance/rustix)                   |              |                    |

Note: We are not going to provide a full set of APIs, because they're for general purposes developing. Only a small subset will be implemented, such as file system, child process, operating system, strings, bytes, data structures, IO, network, etc. And also we may want to remain a small group instead of being super big because it may not be worth it.
