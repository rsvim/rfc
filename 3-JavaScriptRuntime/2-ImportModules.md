# Import Modules

> Written by @linrongbin16, first created at 2025-09-02.

This RFC investigates javascript modules and `import` keyword implementations in [dune](https://github.com/aalykiot/dune). As dune is a (the only) very small code base, but basically complete javascript runtime, developed with rust/tokio and V8 js engine. It is the best example to show what a javascript runtime is doing inside, and fills up all the gaps between js engine and a ready-to-use general purposed language interpreter/virtual-machine, including standard library, `Promise`/`async`/`await` scheduler, etc.
