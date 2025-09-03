# Import Modules

> Written by @linrongbin16, first created at 2025-09-02.

This RFC investigates javascript modules and `import` keyword implementations in [dune](https://github.com/aalykiot/dune).

> Recommend reading: [RFC-2](../2-EventLoop.md)

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

## Pseudo-Code of Initialization

From a javascript source code is read, until it is executed by V8 engine, the basic process is:

1. Read source code file. NOTE: the `import` supports a remote resource such as `import syntax from "https://jsdlr.com/syntax.js";`, in such case, this step becomes download + read. But in this section, let's simply think of all javascript scripts are on local file system.
2. Compile into V8 module.
3. Fetch all static import dependencies, each dependency is also a V8 module. NOTE: Here we can only fetch all static import dependencies (`import ... from ...`), dynamic import need to be fetched during module evaluating (`await import(...)`).
4. Instantiate V8 module with module resolver callback. NOTE: In this step, all the dependencies should be already cached and ready to use.
5. Evaluate (execute) V8 module.

When we run `dune run ./index.js` in the terminal. All modules is a dependency tree:

![1](../images/3-JavaScriptRuntime-2-ImportModules.1.drawio.svg)

In real-world project, the dependencies can be a big ocean, simply loading them can be a challenge. Dune uses a classic architecture to solve this issue (node/deno also use this solution, but more completed): event loop + async task.

An async task has two steps:

1. Work: The task has a job to do. For example:
   - Read file content. This task can use async file IO, thus it can be run within a single thread.
   - Network/http. The socket/network/http is similar to file IO, it can also use async IO.
   - Sleep/timeout. This task can use a timer to calculate how many milliseconds/seconds/hours has elapsed, thus it doesn't block the "main" thread.
   - Some real CPU-bound calculation tasks.
2. Complete: Once the task has done the work, it can trigger the next step with a callback.

All tasks are dispatched to a backend thread-pool to execute, so even the CPU-bound calculation tasks will not block the "main" thread. Once "work" step is done, they will be adding to a `pending_futures` queue, waiting for the "main" thread to call the "complete" steps.

For module resolving, the task is called `EsModuleFuture`, its "work" step is simply reading source code by a file path. While its "complete" step is:

```text
1 If current module has exceptions:
2   Stops current process
3 Else:
4   Compile the source code into V8 module (here we call it the "current" module)
5   Get all its dependency modules from "current" module
6   For each dependency module:
7     Create new `EsModuleFuture` task and push to `pending_futures` queue
```

The event loop (v1) of dune runs in below pseudo-code process:

```text
1 Main:
2   Read arguments from CLI, i.e. the entry file name `index.js`.
3   Initialize js runtime and V8 engine.
5   Create the first `EsModuleFuture` task and push to the `pending_futures` queue.
6   Loop:
7     let `pending_tasks` = Remove all completed tasks from the `pending_futures` queue.
8     For each task in `pending_tasks`:
9       If the task is `EsModuleFuture`, do the "complete" step.
```

NOTE: In line 7, for all the pending tasks, their "work" steps are already completed.

## Module Caches

A module can be a common dependency for many other modules. In event loop (v1), it may load a common dependency many times, duplicatedly. Here comes the module map, it is a hash map that caches a compiled module and avoid duplicated loading.

Before introducing it, let's first define some very common utilities:

```rust
pub type ModulePath = String;
pub type ModuleSource = String;
pub enum ImportKind {
    // Loading static imports.
    Static,
    // Loading a dynamic import.
    Dynamic(v8::Global<v8::PromiseResolver>),
}
pub enum ModuleStatus {
    // Indicates the module is being fetched.
    Fetching,
    // Indicates the dependencies are being fetched.
    Resolving,
    // Indicates the module has ben seen before.
    Duplicate,
    // Indicates the modules is resolved.
    Ready,
}
```

- `ModulePath` indicates the file path of a javascript script file.
- `ModuleSource` indicates the source code content.
- `ImportKind` indicates whether the module is static import or dynamic import.
- `ModuleStatus` indicates current module status:
  - `Fetching`: Initialize status. When a module is first been created, it's status is `Fetching`. It corresponds to line-5 in above pseudo-code process, i.e. create a `EsModuleFuture` and let backend thread-pool to read the source code from the javascript file.
  - `Resolving`: After the source code is read, and compiled into V8 module, but it still has many dependency modules, it's status is `Resolving`. It corresponds to line-13 in above pseudo-code process, i.e. it creates more `EsModuleFuture` tasks for each dependencies.
  - `Duplicate`: When fetching a module, if it is already been loaded before and cached, we can directly mark it as `Duplicate` and skip fetching again.
  - `Ready`: When a module and all its dependency modules are fetched and compiled into V8 modules, it's status is `Ready`.

### Module Graph

A module can have no dependency, or it can have multiple dependencies.

```rust
struct EsModule {
    pub path: ModulePath,
    pub status: ModuleStatus,
    pub dependencies: Vec<Rc<RefCell<EsModule>>>,
    pub exception: Rc<RefCell<Option<String>>>,
    pub is_dynamic_import: bool,
}
struct ModuleGraph {
    pub kind: ImportKind,
    pub root_rc: Rc<RefCell<EsModule>>,
    pub same_origin: LinkedList<v8::Global<v8::PromiseResolver>>,
}
```

`EsModule` holds information for a single module:

- `path`: File path of source code.
- `status`: Module status.
- `dependencies`: All its dependencies.
- `exception`: Any exception happened during resolving.
- `is_dynamic_import`: Whether the "current" module is dynamic import.

`ModuleGraph` holds all promises (`v8::PromiseResolver`) for a single module:

- `kind`: This is similar to EsModule's `is_dynamic_import`. But it can hold a `v8::PromiseResolver` if current module is dynamic import.
- `root_rc`: The `EsModule` itself of current module.
- `same_origin`: All dependencies belongs to "current" module that are dynamic imported.

### Module Map

```rust
struct ModuleMap {
    pub main: Option<ModulePath>,
    pub index: HashMap<ModulePath, v8::Global<v8::Module>>,
    pub seen: HashMap<ModulePath, ModuleStatus>,
    pub pending: Vec<Rc<RefCell<ModuleGraph>>>,
}
```

`ModuleMap` holds all the modules for the whole js runtime:

- `main`: The entry file path, in this section, it is the `index.js` file.
- `index`: Holds all modules' file path and its compiled V8 module.
- `seen`: Holds all modules' file path and its status.
- `pending`: Holds all unresolved modules, i.e. the module status is still not `Ready`.

### Pseudo-Code of Initialization With Module Caches

With module caches, the initialization process can be upgrade to:

```text

```

When js runtime initialize, all the static import modules need to be resolved, i.e. their status are `Ready`. Then the js runtime can finally start to evaluate/execute the module. While all dynamic import modules can be delayed until actual evaluation/execution.
