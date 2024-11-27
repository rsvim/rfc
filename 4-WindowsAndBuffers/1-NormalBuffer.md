# Normal Buffer

> Written by @linrongbin16, first created at 2024-11-21, last modified at 2024-11-27.

This RFC describes the normal buffer that maps the file content in filesystem to the memory inside editor.

There are several read/write operations about normal buffer\[[1](#references)\]:

- `:e[dit] {filename}`\[[2](#references)\]: Edit file, open a new buffer and bind it with a `{filename}`.
- `:e[new]`\[[3](#references)\]: Create a new detached buffer, associated with no file.
- `:file {filename}`\[[4](#references)\]: Bind a buffer to another `{filename}`, i.e. rename a buffer's associated file name.
- `:sav[eas] {filename}`\[[5](#references)\]: Save current buffer contents into another `{filename}`, instead of saving to current associated file.
- `:{,range} {filename}`\[[6](#references)\], `:w[rite] {filename}`\[[7](#references)\] : Save part (selected by line range) of current buffer contents into another `{filename}`, instead of saving to current associated file.

To support these features, normal buffer (it can also be called file buffer because it is mostly presented as a file in filesystem) contains two types of states:

1. Associated (with a file on the file system) or detached (with no file). Note: The _**filesystem**_ can be not only local storage, but also remote via network protocols.

2. Several running status, a certain time point has a certain status:

   - Initialized: When a file buffer is created, it is always detached, and the status is always _**initialized**_.
   - Changed: When buffer contents are different from file system (if associated) or once been modified (if detached). Note: once a detached buffer is been modified, it will always stay in _**changed**_ status, and cannot go back to _**initialized**_ status.
   - Loading: When the buffer reads the content (or meta info) from associated file into the buffer, the status is _**loading**_. Note: a detached buffer cannot change its status to _**loading**_.
   - Saving: When the buffer writes the buffer's contents into the associated file, the status is _**saving**_. Note: a detached buffer cannot change its status to _**saving**_.
   - Synced: When the buffer's loading/saving operation is completed, the status changes to _**synced**_.

The associated-detached state and running status can be switched to each other:

![1](../images/4-WindowsAndBuffers-1-FileBuffer.1.drawio.svg)

Note:

1. `Loading` and `Saving` are momentary states, since Rsvim is designed with the concept of never freezing, file operations are handled with async manner and won't block TUI. During these states, user can still do a lot of operations such as moving cursor, switching to other buffers, etc. But buffer modification is not allowed, i.e. user cannot edit/delete the buffer when it is reading/writing contents from/to the file on the file system. This is mostly to ensure user data security.
2. If a buffer is detached and modified, it is always _**changed**_ and will never go back to _**initialized**_.
3. If a buffer is associated with a non-existing file and modified, it is always _**changed**_ and will never go to _**synced**_ unless it is been saved.

The above flow chart shows the status for only one certain buffer, there is no other buffers in the flow chart. And there still are big gaps between the internal states and the final user facing ex commands (i.e. `:edit`, `:file`, etc), this is only a middle-level design.

## References

- \[1\]: <https://vimhelp.org/editing.txt.html>
- \[2\]: <https://vimhelp.org/editing.txt.html#%3Aedit>
- \[3\]: <https://vimhelp.org/editing.txt.html#%3Aenew>
- \[4\]: <https://vimhelp.org/editing.txt.html#%3Afile_f>
- \[5\]: <https://vimhelp.org/editing.txt.html#%3Asav>
- \[6\]: <https://vimhelp.org/cmdline.txt.html#%3A%2C>
- \[7\]: <https://vimhelp.org/editing.txt.html#%3Awrite_f>
