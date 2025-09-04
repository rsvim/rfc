# Import Modules

> Written by @linrongbin16, first created at 2025-09-02.

This RFC investigates javascript modules and `import` keyword implementations in [dune](https://github.com/aalykiot/dune). Also recommend reading [RFC-2](../2-EventLoop.md).

## Why Dune?

The dune project:

- Is an open source javascript runtime.
- Has a very small code base, but still covers all the most important components/algorithms/soultions of a javascript runtime.
- Is developed with rust/tokio and V8 js engine.
- Is the best example to show what a javascript runtime is doing inside, and how it fills up all the gaps between js engine and a ready-to-use general purposed language interpreter/virtual-machine, including standard library, `Promise`/`async`/`await`, runtime APIs, etc.
- Rsvim copied the architecture and a lot source code from it (big thanks!).

As for other projects, they are either too small, just a "Hello World" tutorial, or too big (i.e. node and deno) to go through the whole code base, no one can quickly understand their core components, design principles and technical solutions.

## Module

### Introduction

A javascript-based project is similar to other programming language project, usually it has two types:

- As an executable file: for example, running `dune run file.js` (or `node file.js`, `deno file.js`) to execute the `file.js` script.
- As a library/package: for example, using `import syntax from "syntax";` in the `file.js` to import a javascript module.

### CommonJS/ECMA Modules

A compiled javascript script file is a "module".

> For more details, please checkout [Node Modules/Packages](https://nodejs.org/api/packages.html). In this section, I will use very simple words to describe the technical terms. They are not complete or accurate, but easy to understand. Welcome to suggest better descriptions.

The `file.js` (in the above example) is also a "module". As the first executable module feed into `dune` command line, it is called the "main module". In node modules, there're two types of module standards:

- [CommonJS modules](https://nodejs.org/api/modules.html#modules-commonjs-modules) (CJS)
- [ECMA modules](https://nodejs.org/api/esm.html) (ESM) defined in [ECMA-262](https://tc39.es/ecma262/#sec-modules)

A CommonJS module is imported by the `require` keyword:

```javascript
const syntax = require("syntax");
```

A ES module is imported by the `import` keyword:

```javascript
import syntax from "syntax";
```

And ES module support 3 variants:

- Static import:

  ```javascript
  // Import without side-effects
  import syntax from "syntax";

  // Import with side-effects
  import "syntax";
  ```

- Dynamic import:

  ```javascript
  // Import asynchronously
  const syntax = await import("syntax");

  // Or
  import("syntax").then((syntax) => {}).catch((err) => {});
  ```

The word "side-effects" is to describe there are actual impacts and changes been made to the Operating Systems. It can be print messages to console/terminal, read/write files in file system, receive/send data via IPC/RCP to remote systems, etc. When we start a program, we usually expect it does some "side-effects" to help us. But for library/package, we usually expect they don't have any side-effects, just expose some utilities and let the executable files call them.

In this section, we can simply think of "static import" and "commonjs require" are the same thing.

### A Real-World Node Project with NPM Packages

#### Step-1 Initialize Project

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

- `index.js` as a main module, it can be executed by `dune run ./index.js` and print a "Hello" message (or `node ./index.js`, `deno ./index.js`).
- `util.js` as a library module, it implements many fundamental utilities and doesn't have any side-effects.

#### Step-2 Add NPM Packages

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

Then let's edit the `index.js` file, change it to:

```javascript
import * as util from "./util.js";
import react from "react"; // This line is unused

util.hello();
```

And changes the `package.json` file to:

```json
{
  "type": "module",
  "dependencies": {
    "@mui/material": "^7.3.2"
  }
}
```

This is because `node` by default uses CommonJS modules, but in this section we will focus on ES modules. Adding `"type": "module"` will let `node` uses ES modules.

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

- A npm package can have multiple files, thus the code logic is more fine-grained controlled.
- A module can be specified with relative file path started with `./` or `../`, or full file path started with `/` or `C:\\` (on Windows).
- A module can be specified with a npm package name. Node looks for the entry module by the [`"exports" entry points`](https://nodejs.org/api/packages.html#package-entry-points) specified inside `package.json`, usually it is the `index.js` file. For example, the "react" `package.json` is:

  ```json
  {
    ...
    "exports": {
      ".": {
        "react-server": "./react.react-server.js",
        "default": "./index.js"
      },
      ...
    },
    ...
  }
  ```

  Thus when we write `import react from "react";`, it actually import the `react/index.js` file.

- A `package.json` file specifies all dependencies required by "current" package, thus npm can install all of them for user.

## Resolve Module

Inside js runtime, it needs a few steps to evaluate (execute) a module:

1. Read source code from file path.

   > NOTE: The `import` keyword supports a remote resource. For example `import react from "https://cdn.jsdelivr.net/npm/react@19.1.1/cjs/react.production.min.js";`. In such case, this step requires extra downloading resource from http/network. In this section, let's simply ignore them to keep it simple.

2. Compile source code into V8 module.
3. Recursively resolve all static imported dependencies, each dependency is also a module. NOTE: dynamic imported modules need to be resolved during evaluating (executing) current module.
4. Instantiate V8 module with a module lookup callback function. In this step, all dependencies of current module should be already resolved and ready to use.
5. Evaluate (execute) V8 module.

When we run `dune run ./index.js` in the terminal. All modules is a dependency tree:

![1](../images/3-JavaScriptRuntime-2-ImportModules.1.drawio.svg)

### [`JsFuture`](https://github.com/aalykiot/dune/blob/8f61719c7765d371e4f77ee3a4cf9d82e59391e7/src/runtime.rs?plain=1#L44)

Js runtime such as node/deno is famous for their "async event loop", which brings a great performance. The "async" is implemented by several core components:

- Async IO: Most IO operations are running asynchronously to avoid blocking, for example file IO, standard IO (`stdin`, `stdout`, `stderr`), socket, network, http, ssl/tls, etc.
- Thread Pool: For CPU-bound tasks, they are spawn with a worker thread to avoid blocking the main thread.
- Callback: For each async task, once its work is completed, its callback consumes the work result and finally completes itself.

All async tasks share a same trait: `JsFuture`:

```rust
pub trait JsFuture {
    fn run(&mut self, scope: &mut v8::HandleScope);
}
```

Resolving a module is also one of async tasks.

### Common Types

Before introducing module resolving, here're some common types in dune:

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

### [`EsModule`](https://github.com/aalykiot/dune/blob/8f61719c7765d371e4f77ee3a4cf9d82e59391e7/src/modules.rs?plain=1#L153)

In real-world project, the dependencies can be a big ocean, simply loading them is a big challenge. The event loop handles all the module resolving tasks along with all the async tasks together. Scheduled in 3 steps:

1. Spawn with a worker, i.e. the actual work will be done without blocking "main" thread. Then push the task to a `pending_futures` queue.
2. Wait for the task work done.
3. Run the callback with work results, remove from `pending_futures` queue.

Module resolving task is called `EsModuleFuture`, its "work" step is simply reading source code by a file path. Its "callback" step is:

```text
If current module has exceptions:
    If it is static import, then stop the whole process
Else:
    Compile the source code into V8 module
    Get all its dependencies from current module
    For each dependency module:
        Create new `EsModuleFuture` task and push to `pending_futures` queue
```

In the "callback" step, if there's any error, `EsModuleFuture` will set an exception for this module.

Here is the rust version `EsModule`:

```rust
struct EsModule {
    pub path: ModulePath,
    pub status: ModuleStatus,
    pub dependencies: Vec<Rc<RefCell<EsModule>>>,
    pub exception: Rc<RefCell<Option<String>>>,
    pub is_dynamic_import: bool,
}
```

`EsModule` holds information for a single module:

- `path`: File path of source code.
- `status`: Module status.
- `dependencies`: All its dependencies. A module can have no dependency, or it can have multiple dependencies.
- `exception`: Any exception happened during resolving.
- `is_dynamic_import`: Whether the "current" module is dynamic import.

### [`ModuleGraph`](https://github.com/aalykiot/dune/blob/8f61719c7765d371e4f77ee3a4cf9d82e59391e7/src/modules.rs?plain=1#L208)

The rust version `ModuleGraph` is:

```rust
struct ModuleGraph {
    pub kind: ImportKind,
    pub root_rc: Rc<RefCell<EsModule>>,
    pub same_origin: LinkedList<v8::Global<v8::PromiseResolver>>,
}
```

`ModuleGraph` holds all promises (`v8::PromiseResolver`) for a `EsModule`:

- `kind`: This is similar to EsModule's `is_dynamic_import`. But it can hold a `v8::PromiseResolver` if current module is dynamic import.
- `root_rc`: The `EsModule` of current module.
- `same_origin`: All dependencies belongs to "current" module that are dynamic imported.

### [`ModuleMap`](https://github.com/aalykiot/dune/blob/8f61719c7765d371e4f77ee3a4cf9d82e59391e7/src/modules.rs?plain=1#L81)

A module can be a common dependency for many other modules. When event loop runs, it may load a common dependency many times, duplicatedly. A module caches (a hash map) can cache all compiled modules and avoid duplicated loading.

Here's the rust version `ModuleMap`:

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

### Pseudo-Code Process

Finally, the process of event loop written in pseudo-code is:

```text
1  Main:
       Initialize V8 engine.
       Initialize `module_map` (ModuleMap).
       Initialize `pending_futures` queue.
5      Create first `EsModuleFuture` task and push to the `pending_futures` queue.
       Loop:
           (Step-1) Fast forward imports:
           |   For each pending module in `module_map.pending` queue:
           |       If current module has any exception while resolving:
10         |           Assert it must be dynamic import, reject the promise.
           |           Remove it from `module_map.pending` queue.
           |       Else:
           |           If current module is not `Ready` yet:
           |               If current module is `Duplicate`:
15         |                   Mark it as `Ready`.
           |               For all its dependencies of current module:
           |                   Fast import for one dependency:
           |                   |   If it is already `Ready`:
           |                   |       Do nothing
20         |                   |   If it is `Duplicate`:
           |                   |       Mark it as `Ready`
           |                   |   If it has no dependencies, and it's `Resolving`:
           |                   |       Mark it as `Ready`
           |                   |   If it has no dependencies, and it's not `Resolving`:
25         |                   |       Do nothing
           |                   |   If all the dependencies of it are `Ready`:
           |                   |       Mark it as `Ready`
           |                   If current module is `Ready`:
           |                       Push it to a `ready_imports` queue.
30         |                       Remove it from `module_map.pending` queue.
           | For each module in `ready_imports` queue:
32         |   Compile source code into v8 module.
           |   Evaluate (execute) it.
           |   If it is dynamic import and there's exception:
35         |       Reject the promise of this dynamic import
           (Step-2) Tick the event loop (We don't explain this part in this section)
           (Step-3) Run all the callbacks waiting in `pending_futures` queue:
           |   For each callback in `pending_futures` queue:
39         |       Run the callback.
```

The whole process is quite long, the most important two lines are:

- For line 32, the "main" module will be the first pushed to `module_map.pending`, but the last to evaluate/execute at this line. When js runtime initializes, all static imported modules should to be resolved, i.e. their status are `Ready`. Then the js runtime can finally start to evaluate/execute the module. While all dynamic import modules can be delayed until actual evaluation/execution.
- For line 39, if the callback is a `EsModuleFuture`, it will compile source code into v8 module, and creates tasks for all dependencies (as we mentioned in [`EsModule`](#esmodule)).
