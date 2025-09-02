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

The `file.js` (in the above example) is also a "module". As the first executable module feed into `dune` command line, it is called the "main module". In node modules, there're two types of module standards: [CommonJS modules](https://nodejs.org/api/modules.html#modules-commonjs-modules) and [ECMA modules](https://nodejs.org/api/esm.html) defined in [ECMA-262](https://tc39.es/ecma262/#sec-modules).

A common js module is imported by the `require` keyword:

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
