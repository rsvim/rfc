# Import Module

> Written by @linrongbin16, first created at 2024-09-26, last updated at 2024-09-27.

This RFC describe how the modules resolved and imported in the js runtime.

## Static Import

Keyword `require` and `import` (we will discuss _**dynamically import**_ later) are static ways to import modules in javascript language. When this comes to the RSVIM editor, all modules are simply external config files that extracted from the main `.rsvim.{js,ts}` user config file, they can be plugins as well. For each module, it will be evaluated only once.

A confliction we have is: ECMAScript standards support [import modules from URL](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules), i.e. a js module can be directly downloaded from network during it's been executed. For example:

```javascript
import { name } from "https://example.com/shapes/circle.js";
```

This behavior is quite different with the VIM editor, because VIM editor always first download all plugins onto local file system, then load them on editor's startup, which runs in a completely sequential order. Once we implement the `import` keyword in async way, it means every time we use `import`, the event loop has to resolve the module in next tick.

Here we have several things to take consideration in this scenario:

1. Design: Does user really need this feature?
2. Security: Is it safe if we allow the editor directly downloading and loading plugins from network/http?
3. Behavior: Is the behavior still remains consistent if the editor first start TUI and let user use it, then change its behavior after the module completes the plugin downloading and loading?
4. Compatible: Do we need to stick to ECMAScript standard to be more compatible with js language?

For now, we follows the tradition behavior of the editor, i.e. we don't support the URL import in js module, all external modules need to be downloaded before editor loading them. For more discussion, we may leave for the future.

## Dynamically Import

ECMAScript standard also support the [dynamic import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import):

```javascript
import("/my-module.js").then((mod2) => {
  console.log(mod === mod2); // true
});
```

In this way, it's actually a `Promise` and must be async, i.e. the `then` code block will have to run in next tick until the module is been resolved.
