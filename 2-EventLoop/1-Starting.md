# Starting

This RFC describes the starting (initialization) logic of the editor, only for normal TUI mode, other modes are not covered.

The editor runs steps on start up:

1. Set `shell` options.
2. Process command line arguments.
   1. Load input files into buffers.
   2. Execute the `-c` arguments after the first file is loaded, before loading any configs.
   3. Execute the `--cmd` arguments after all input files are loaded, before loading any configs.
3. Load configs, i.e. the `vimrc` file (for Vim) or `init.lua` file (for Neovim).
   1. Load the config entry file (say `.rsvim.js` or `.rsvim.ts`).
   2. For all the js/ts modules specified with `require` and `import` keywords, continue to load them recursively.

## Step-2 Process Command Line Arguments

Here are some edge cases worth to discuss:

### Data Racing in Buffers between User Editing and File Loading

If the 1st input file is quite large and need some minutes to read into buffers completely. There will be a data racing issue:

1. The default window and buffer is created immediately after editor start, and allow user editing the empty buffer.
2. The 1st file is still been reading into buffer, and not ready for editing.

Here use a chunks reading method:

1. When editor start, it creates the default window and an empty buffer binding with it.
2. Input files will be loaded in blocks per 4096 bytes, and sync back to the buffer after reading a block done.

## References

- [Neovim user documentation - Initialization](https://neovim.io/doc/user/starting.html#_initialization)
- [Vim help - Initialization](https://vimhelp.org/starting.txt.html#initialization)
