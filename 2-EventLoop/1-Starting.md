# Starting

This RFC describes the starting (initialization) logic of the editor, only for normal TUI mode, other modes are not covered.

The editor runs steps on start up:

1. Set `shell` options.
2. Process command line arguments.
   1. Load input files into buffers.
   2. Execute the `--cmd` arguments (before loading any configs).
3. Load configs, i.e. the `vimrc` file (for Vim) or `init.lua` file (for Neovim).

## Step-2 Process Command Line Arguments

Here are some edge cases worth to discuss:

### Data Racing in Buffers between User Editing and File Loading

## References

- [Neovim user documentation - Initialization](https://neovim.io/doc/user/starting.html#_initialization)
- [Vim help - Initialization](https://vimhelp.org/starting.txt.html#initialization)
