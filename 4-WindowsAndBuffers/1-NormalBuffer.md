# Normal Buffer

> Written by @linrongbin16, first created at 2024-11-21, last modified at 2024-11-27.

This RFC describes the normal buffer that maps the file content in filesystem to the memory inside editor. Here we propose two manners for data syncing between buffers and filesystems: async and sync.

## States

There are several read/write operations about normal buffer\[[1](#references)\]:

- `:e[dit] {filename}`\[[2](#references)\]: Open file and read the contents into a new buffer, set the `{filename}` for the buffer.
- `:e[new]`\[[3](#references)\]: Create a new detached buffer, associated with no file.
- `:file {filename}`\[[4](#references)\]: Set another `{filename}` for the buffer, i.e. rename a buffer's associated file name.
- `:sav[eas] {filename}`\[[5](#references)\]: Save current buffer contents into another `{filename}`, instead of saving to current associated file.
- `:{,range} {filename}`\[[6](#references)\], `:w[rite] {filename}`\[[7](#references)\] : Save part (selected by line range) of current buffer contents, or all of it, into another `{filename}`, instead of saving to current associated file.

To support these features, normal buffer (it can also be called file buffer because it is mostly presented as a file in filesystem) contains two types of states:

1. Associated (with a file on the filesystem) or detached (with no file). Note: The _**filesystem**_ can be not only local storage, but also remote via network protocols.

2. Several running status, a certain time point has a certain status:

   - Initialized: When a detached buffer is created, the status is always _**initialized**_.
   - Changed: When buffer contents are different from filesystem (if associated) or once been modified (if detached). Note: once a detached buffer is been modified, it will always stay in _**changed**_ status, and cannot go back to _**initialized**_ status.
   - Loading: When the buffer reads the content (or meta info) from associated file into the buffer, the status is _**loading**_. Note: a detached buffer cannot change its status to _**loading**_.
   - Saving: When the buffer writes the buffer's contents into the associated file, the status is _**saving**_. Note: a detached buffer cannot change its status to _**saving**_.
   - Synced: When the buffer's loading/saving operation is completed, the status changes to _**synced**_.

The associated-detached state and running status can be switched to each other:

![1](../images/4-WindowsAndBuffers-1-FileBuffer.1.drawio.svg)

Note:

1. `Loading` and `Saving` are momentary states, since Rsvim is designed with the concept of never freezing, we would want all IO operations are handled with async manner and won't block TUI. During these states, user can still do a lot of operations such as moving cursor, switching to other buffers, etc. But buffer modification is not allowed, i.e. user cannot edit/delete the buffer when it is reading/writing contents from/to the file on the file system. This is mostly to ensure user data security.
2. If a buffer is detached and modified, it is always _**changed**_ and will never go back to _**initialized**_.
3. If a buffer is associated with a non-existing file and modified, it is always _**changed**_ and will never go to _**synced**_ unless it is been saved.

The above flow chart shows the status for only one certain buffer, there is no other buffers in the flow chart. And there still are big gaps between the internal states and the final user facing ex commands (i.e. `:edit`, `:file`, etc), this is only a middle-level design.

## Multiple-Threading and Async IO

When implementing the buffer's operation primitives in a multiple-threading and async environment, we would want each primitives are thread-safe and atomic to higher-level. For example, now we have such a primitive, or say, a buffer API:

- `OpenFile`: This API opens file (by argument `{filename}`) with a new buffer, there are two cases it needs to handle:
  - If the file exists, it reads both the file's metadata (create time, last modified time, is symbolink, etc) and file contents into the newly created buffer, and also saved the last sync time to _**current time**_.
  - If the file doesn't exist, it simply saves the `{filename}`, the set the last sync time to `None`.

Below is the pseudocode to describe the logic:

```text
Create new buffer.
Set buffer status to `Loading`.
Set buffer file name to `{filename}`.
Set buffer metadata to `None`.
Set buffer last sync time to `None`.

Open the file.
  if successful:
    Read the file metadata.
    if read metadata is successful:
      Set buffer metadata.
    else:
      Delete the buffer.
      Returns the IO error.

    Read the file content.
    if read content is successful:
      Set buffer content.
    else:
      Delete the buffer.
      Returns the IO error.

    Set buffer status to `Synced`.
    Set buffer last sync time to `Now`.
    Returns success
  else:
    if error is "File not found":
      Set buffer status to `changed`.
      Set buffer file name to `{filename}`.
      Returns success
    else:
      Delete the buffer.
      Returns the IO error.
```

As we could see, there are actually several OS-level file IO operations happened and multiple buffer internal data changed inside this API.

When in a multiple-threading and async runtime, this API runs in an async manner. Thus other threads (especially the logics running in javascripts) could still try to fetch this buffer's data (I didn't mean it must happens with `OpenFile` API we used as an example here, but with the developing of Rsvim, there will be more and more primitives, so one day this will happen). It will eventually lead to issues like the data synchronization in database transactions, or like the data racing issue in multiple-threading runtime.

With async IO, we would have to support a full featured transactional mechanism with the ACID (atomicity, consistency, isolation, durability) properties, or buffer-level mutex/condition/notify mechanism. Both solutions are not going to be easy.

## Single-Threading and Sync IO

But if we simply use single-threading and sync IO to synchronize data between buffer and filesystem, there will be no `Loading` and `Saving` status, and javascripts layer will have a much more easy running environment. The internal dataflow becomes:

![2](../images/4-WindowsAndBuffers-1-FileBuffer.2.drawio.svg)

## References

- \[1\]: <https://vimhelp.org/editing.txt.html>
- \[2\]: <https://vimhelp.org/editing.txt.html#%3Aedit>
- \[3\]: <https://vimhelp.org/editing.txt.html#%3Aenew>
- \[4\]: <https://vimhelp.org/editing.txt.html#%3Afile_f>
- \[5\]: <https://vimhelp.org/editing.txt.html#%3Asav>
- \[6\]: <https://vimhelp.org/cmdline.txt.html#%3A%2C>
- \[7\]: <https://vimhelp.org/editing.txt.html#%3Awrite_f>
