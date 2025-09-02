# Import Modules

> Written by @linrongbin16, first created at 2025-09-02.

This RFC investigates javascript modules and `import` keyword implementations in [dune](https://github.com/aalykiot/dune).

## Why Dune?

The dune project:

- Is an open source javascript runtime.
- Has a very small code base, but still covers almost covers all the most important components as a javascript runtime.
- Is developed with rust/tokio and V8 js engine, shared exactly the same technical stack with Rsvim.
- Is the best example to show what a javascript runtime is doing inside, and how it fills up all the gaps between js engine and a ready-to-use general purposed language interpreter/virtual-machine, including standard library, `Promise`/`async`/`await` scheduler, etc.

As for other projects, they are either too small, just a "Hello World" tutorial, or too big to quickly go through the whole code base, we can no longer quickly understand their core components, design principles and technical solutions (i.e. node/deno).

## JavaScript-based Project

A javascript-based project is similar to other programming language project, mostly it has two types:

- As an executable file: for example, running `dune file.js` (similar to `node file.js`, `deno file.js`) to execute the `file.js` script.
- As a library/package: for example, using `import syntax from "syntax";` (inside "current" js script) to import a javascript module to avoid duplication and manual copy-pasting.

## Module

Based on [Node Modules/Packages](https://nodejs.org/api/packages.html) document, a compiled javascript script file is a "module".

> NOTE: In this section, I will use very simple words to describe the technical terms. They are not complete or accurate, but easy to understand. Welcome to suggest better words/descriptions.

The `file.js` (in the above example) is also a "module". As the first executable module feed into `dune` command line, it is called the "main module". Now let's extend the `file.js` single file module to a multi-file project named "syntax", the file structure is:

```text
syntax/
|- index.js
|- util.js
```

`index.js` is:

```javascript
import * as util from "./util.js";

util.hello();
```

`util.js` is:

```javascript
export function hello() {
  console.log("Hello");
}
```
