# Import Modules

> Written by @linrongbin16, first created at 2025-09-02.

This RFC investigates javascript modules and `import` keyword implementations in [dune](https://github.com/aalykiot/dune).

## Dune

The dune project:

- Is an open source javascript runtime.
- Has a very small code base, but still covers almost covers all the most important components as a javascript runtime.
- Is developed with rust/tokio and V8 js engine, shared exactly the same technical stack with Rsvim.
- Is the best example to show what a javascript runtime is doing inside, and how it fills up all the gaps between js engine and a ready-to-use general purposed language interpreter/virtual-machine, including standard library, `Promise`/`async`/`await` scheduler, etc.

As for node and deno, they are just too big to go through the code base, we can no longer quickly understand their core components, design principles and technical solutions.
