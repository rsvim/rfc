# Async

> Written by @linrongbin16, first created at 2024-09-26.

This RFC describe how the event loop support the async functions and callbacks inside javascript code.

## Import (Module)

Keyword `require`, `import` and dynamically import are 3 ways to import modules in javascript language. For js (actually for most generic purpose scripting languages), a module can do:

1. Export functions, classes, global variables, etc as common utilities for others.
2. Directly execute some logics.

And a module will be evaluated only once.

> Note: When this comes to the RSVIM editor, all modules are simply external config files that extracted from the main `.rsvim.{js,ts}` user config file. And they can be plugins too.

A confliction we have is: ECMAScript standards support [directly import modules from a URL](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules), i.e. a js module can be directly downloaded from the internet during it's been executed. For example:

```javascript
import { name } from "https://example.com/shapes/circle.js";
```

This behavior is quite different with the VIM editor, because VIM editor always first download all plugins onto local file system, and load them on editor's startup. Here we have two things to take consideration in this scenario:

1. Security: Is it safe if we allow the editor directly download external plugins and execute it?
2. Behavior: Is the behavior still remains consistent if the editor first start and work for users, then change its behavior after the module completes its downloading and loading?

For now, we follows the tradition behavior of the editor, i.e. we don't support the URL import in js module, all external modules need to be downloaded before editor loading them.
