# Import Modules

> Written by @linrongbin16, first created at 2025-09-02.

This RFC investigates javascript modules and `import` keyword implementations in [dune](https://github.com/aalykiot/dune).

## Why Dune?

The dune project:

- Is an open source javascript runtime.
- Has a very small code base, but still covers almost covers all the most important components as a javascript runtime.
- Is developed with rust/tokio and V8 js engine, Rsvim copied the architecture and a lot source code from it. Big thanks!
- Is the best example to show what a javascript runtime is doing inside, and how it fills up all the gaps between js engine and a ready-to-use general purposed language interpreter/virtual-machine, including standard library, `Promise`/`async`/`await` scheduler, etc.

As for other projects, they are either too small, just a "Hello World" tutorial, or too big to quickly go through the whole code base, we can no longer quickly understand their core components, design principles and technical solutions (i.e. node/deno).

## JavaScript-based Project

A javascript-based project is similar to other programming language project, mostly it has two types:

- As an executable file: for example, running `dune file.js` (similar to `node file.js`, `deno file.js`) to execute the `file.js` script.
- As a library/package: for example, using `import syntax from "syntax";` (inside "current" js script) to import a javascript module to avoid duplication and manual copy-pasting.

## Module

A compiled javascript script file is a "module".

> For more details about modules, please checkout [Node Modules/Packages document](https://nodejs.org/api/packages.html). In this section, I will use very simple words to describe the technical terms. They are not complete or accurate, but easy to understand. Welcome to suggest better words/descriptions.

The `file.js` (in the above example) is also a "module". As the first executable module feed into `dune` command line, it is called the "main module". In node modules, there're two types of module standards: [CommonJS modules](https://nodejs.org/api/modules.html#modules-commonjs-modules) (CJS) and [ECMA modules](https://nodejs.org/api/esm.html) (ESM) defined in [ECMA-262](https://tc39.es/ecma262/#sec-modules).

A CommonJS module is imported by the `require` keyword:

```javascript
const syntax = require("syntax");
```

A ES module is imported by the `import` keyword:

```javascript
import syntax from "syntax";
```

## A Real-World Node Project with NPM Packages

Now let's extend a single file module into a multi-file project named "syntax", the file structure is:

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

- `index.js` as a main module, it can be executed by `dune run ./index.js` and print a "Hello" message (You can also run `node ./index.js` with node).
- `util.js` as a library module, it implements many fundamental utilities and doesn't have any side-effects.

The example is quit simple and easy, but when comes to real-world project, the complexity grows fast and multi-file structure is urgently needed. And here comes the [NPM packages](https://docs.npmjs.com/about-packages-and-modules), which is the most popular and widely used standard for server side javascript-based runtime.

Let's run below command in the "syntax" project:

```bash
npm install @mui/material
```

> NOTE: The `@mui/material` packages have nothing to do with Rsvim, here I just want to demo how real-world NPM packages are structured.

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
    "@mui/material": "^7.3.2"
  }
}
```

The `npm` fetches latest version of packages (including all the dependencies), and download them in `node_modules` directory:

```text
syntax/
|- node_modules/
|  |- .bin/
|  |- @babel/
|  |- @emotion/
|  |- @mui/
|  ...
|- index.js
|- package-lock.json
|- package.json
|- util.js
```

Let's check the `Accordion.js` file inside `@mui`:

```text
syntax/
|- node_modules/
|  |- .bin/
|  |- @babel/
|  |- @emotion/
|  |- @mui/
|  |  |- core-downloads-tracker/
|  |  |- material/
|  |  |  |- Accordion/
|  |  |  |  |- Accordion.d.ts
|  |  |  |  |- Accordion.js
|  |  |  |  ...
|  |  |- utils/
|  |  |  |- chainPropTypes/
|  |  |  |- composeClasses/
|  |  |  ...
|  |- react/
|  |  |  |- cjs/
|  |  |  |- LICENSE
|  |  |  |- README.md
|  |  |  |- compiler-runtime.js
|  |  |  |- index.js
|  |  |  ...
|  |- react-dom/
|  |- react-is/
|  |  ...
|  |- .package-lock.json
|- index.js
|- package-lock.json
|- package.json
|- util.js
```

In the `Accordion.js` file, it is:

```javascript
"use strict";
"use client";

var _interopRequireDefault =
  require("@babel/runtime/helpers/interopRequireDefault").default;
var _interopRequireWildcard =
  require("@babel/runtime/helpers/interopRequireWildcard").default;
Object.defineProperty(exports, "__esModule", {
  value: true,
});
exports.default = void 0;
var React = _interopRequireWildcard(require("react"));
var _reactIs = require("react-is");
var _propTypes = _interopRequireDefault(require("prop-types"));
var _clsx = _interopRequireDefault(require("clsx"));
var _chainPropTypes = _interopRequireDefault(
  require("@mui/utils/chainPropTypes"),
);
var _composeClasses = _interopRequireDefault(
  require("@mui/utils/composeClasses"),
);
var _zeroStyled = require("../zero-styled");
var _memoTheme = _interopRequireDefault(require("../utils/memoTheme"));
var _DefaultPropsProvider = require("../DefaultPropsProvider");
var _Collapse = _interopRequireDefault(require("../Collapse"));
var _Paper = _interopRequireDefault(require("../Paper"));
```

Let's go through these "require" statements:

- In line 4-7, `require("@babel/runtime/helpers/...")` imports the [`@babel/runtime`](https://www.npmjs.com/package/@babel/runtime) package. NOTE: `@babel/runtime` is a [scoped package](https://docs.npmjs.com/cli/v11/using-npm/scope), i.e. `@babel` is the organization/namespace for all its packages.
- In line 12, `require("react")` imports the [`react`](https://www.npmjs.com/package/react) package.
- In line 13, `require("react-is")` imports the [`react-is`](https://www.npmjs.com/package/react-is) package.
- In line 14, `require("prop-types")` imports the [`prop-types`](https://www.npmjs.com/package/prop-types) package.
- In line 15, `require("clsx")` imports the [`clsx`](https://www.npmjs.com/package/clsx) package.
- In line 16-21, `require("@mui/utils/...")` imports the [`@mui/utils`](https://www.npmjs.com/package/@mui/utils) package.
- In line 22-26, `require("../zero-styled")` and others imports the local modules in upper directory.

As you can see, npm package solves several issues for node/npm:

- Split a npm package into multi-file structure, thus the code logic is more fine-grained controlled.
- The module can be specified with relative file path started with `./` or `../`, or full file path started with `/` or `C:\\` (on Windows).
- The module can be specified with a package name. Node will look for the entry module by the [`"exports" entry points`](https://nodejs.org/api/packages.html#package-entry-points) specified inside `package.json`, usually it is the `index.js` file.
- The `package.json` file specifies all the dependencies used by "this" package, thus npm can install all of them for user.

## Async Import

Recall the "CommonJS modules" and "ECMA modules", the `require` keyword from CommonJS is always synchronous, the `import` keyword from ECMA has 3 use cases:

- Static import (no side-effects): It only imports package as a library and doesn't execute it. For example:

  ```javascript
  import syntax from "syntax";
  ```

- Static import (has side-effects): It imports the library and also execute it, i.e. if there's a `console.log("hello");` inside the library, it will be print. For example:

  ```javascript
  import "syntax";
  ```

- Dynamic import (no side-effects): It imports the library and doesn't execute it, the loading runs asynchronously. For example:

  ```javascript
  const syntax = await import("syntax");
  ```

## V8 Js Engine

From a javascript source code is read, until it is executed by V8 engine, a basic process is:
