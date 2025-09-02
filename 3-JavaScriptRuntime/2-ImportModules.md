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

A compiled javascript script file is a "module".

> For more details about modules, please checkout [Node Modules/Packages document](https://nodejs.org/api/packages.html). In this section, I will use very simple words to describe the technical terms. They are not complete or accurate, but easy to understand. Welcome to suggest better words/descriptions.

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

Here we have 2 modules:

- `index.js` as a main module, it can be executed by `dune` and print a message to `stdout`.
- `util.js` as a library module, it implements many fundamental utilities and doesn't have any side-effects.

The example is quit simple and easy, but when comes to real-world project, the complexity grows fast and multi-file structure is urgently needed. And here comes the [NPM packages](https://docs.npmjs.com/about-packages-and-modules), which is the most popular and widely used standard for server side javascript-based runtime.

Let's run below command in the "syntax" project:

```bash
npm install @mui/material @emotion/react @emotion/styled
```

> NOTE: The `@mui/material`, `@emotion/react`, `@emotion/styled` packages have nothing to do with Rsvim, here I just want to demo how real-world NPM packages are structured.

We will have two new files `package.json`, `package-lock.json` and a new directory `node_modules/`. The file structure becomes:

```text
syntax/
|- node_modules/
|- index.js
|- package-lock.json
|- package.json
|- util.js
```

The `package.json` is:

```json
{
  "dependencies": {
    "@emotion/react": "^11.14.0",
    "@emotion/styled": "^11.14.1",
    "@mui/material": "^7.3.1"
  }
}
```

The `npm` fetches latest version of these three packages (including all their dependencies), and download them in `node_modules` directory:

```text
syntax/
|- node_modules/
|  |- .bin/
|  |- @babel/
|  |- @emotion/
|  ...
|- index.js
|- package-lock.json
|- package.json
|- util.js
```

Let's expand the first package `@babel`, the file structure is:

```text
syntax/
|- node_modules/
|  |- .bin/
|  |- @babel/
|  |  |- code-frame/
|  |  |  |- lib/
|  |  |  |  |- index.js
|  |  |  |  |- index.js.map
|  |  |  |- LICENSE
|  |  |  |- README.md
|  |  |  |- package.json
|  |  |- generator/
|  |  |  |- lib/
|  |  |  |  |- generators/
|  |  |  |  |- node/
|  |  |  |  |- buffer.js
|  |  |  |  |- buffer.js.map
|  |  |  |  |- index.js
|  |  |  |  |- index.js.map
|  |  |  |  ...
|  |  |  |- LICENSE
|  |  |  |- README.md
|  |  |  |- package.json
|  |  ...
|  |- @emotion/
|  ...
|- index.js
|- package-lock.json
|- package.json
|- util.js
```
