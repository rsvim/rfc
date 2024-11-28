# Normal Buffer

> Written by @linrongbin16, first created at 2024-11-21, last modified at 2024-11-28.

This RFC describes the normal buffer that maps the file content from filesystem to the memory managed by editor.

Note: the normal buffer can also be called file buffer because it is mostly presented as a file in filesystem.

Here we propose two ways of data syncing between the in-memory buffers and files on filesystems: async and sync. Finally we choose the sync way to implement this to avoid data racing issue, but it is still worth to document both of the design works.

## Background

There are several read/write operations about normal buffer\[[1](#references)\]:

- `:e[dit] {name}`\[[2](#references)\]: Open file and read the contents into a new buffer, set the `{name}` for the buffer.
- `:e[new]`\[[3](#references)\]: Create a new detached buffer, associated with no file.
- `:file {name}`\[[4](#references)\]: Set another `{name}` for the buffer, i.e. rename a buffer's associated file name.
- `:sav[eas] {name}`\[[5](#references)\]: Save current buffer contents into another `{name}`, instead of saving to current associated file.
- `:{,range} {name}`\[[6](#references)\], `:w[rite] {name}`\[[7](#references)\] : Save part (selected by line range) of current buffer contents, or all of it, into another `{name}`, instead of saving to current associated file.

And since javascript is the first-class citizen in Rsvim editor, these Vim ex commands will be implemented with javascripts, core APIs will be exposed to javascript world from rust side. Image we have below javascript APIs (written in typescript with better types):

```typescript
// List all buffers (by ID).
// It returns all buffer IDs.
// Equivalent to Vim's `:buffers` and Neovim's `vim.api.nvim_list_bufs`.
Rsvim.buf.listBuffers(opts?:{includeUnlist:boolean?}): Array<number>;

// Edit file with a new buffer.
// It returns the newly created buffer ID.
// Equivalent to Vim's `:edit`, similar to Neovim's `vim.api.nvim_create_buf` (it only creates a new buffer, but not opens a file).
Rsvim.buf.editFile(opts:{fileName:string}): number;

// Set file name to a buffer.
// Equivalent to Vim's `:file {name}` and Neovim's `vim.api.nvim_buf_set_name`.
Rsvim.buf.setName(bufferId: number, fileName:string): void;

// Save buffer's contents into its associated file.
// It returns `true` if successful, `false` if there's any error occurred.
// Equivalent to Vim's `:w[rite]` (Neovim doesn't have any equivalent lua APIs).
Rsvim.buf.saveFile(bufferId: number, force?: boolean): boolean;
```

Also please keep in mind: javascript code runs in single-threading with Promise/async/await support, but without any concepts of mutex/lock.

## Async Way

The biggest benefit of the async way is: it can fully utilize the async and multiple-threading environment provided by tokio runtime, and run all IO operations in async mode, thus it can never block the TUI. Everything sounds great until we met the data racing issue.

### States

To support the requirements, normal buffer has two kind of internal states:

1. Associated (or say, named, maps to a file on the filesystem) or detached (or say, unnamed, maps to no file). Note: The _**filesystem**_ can be not only local storage, but also remote via network protocols.

2. Several running status, a buffer in a certain time point has a certain status:

   - Initialized: When a detached buffer is created, the status is always _**initialized**_.
   - Loading: When the buffer reads the contents from the associated file, the status is _**loading**_. Note: a detached buffer cannot change its status to _**loading**_.
   - Saving: When the buffer writes its contents into the associated file, the status is _**saving**_. Note: a detached buffer cannot change its status to _**saving**_.
   - Synced: When the buffer's loading/saving operation is completed, the status changes to _**synced**_, i.e. the buffer's contents are same with the filesystem. Note: since a detached buffer cannot change its status to _**loading**_ or _**saving**_, thus it cannot change to _**synced**_ neither.
   - Changed: When buffer's contents are different from the filesystem:
     - If the buffer is associated with a file name, and its contents are modified since last synced with filesystem, the status is _**changed**_.
     - If the buffer is detached, once it is modified, the status will always stays in _**changed**_ and cannot go back to _**initialized**_.

Here's a flow chart to show how it works on a certain buffer:

![1](../images/4-WindowsAndBuffers-1-FileBuffer.1.drawio.svg)

Note:

1. `Loading` and `Saving` are momentary status. During these status, user can still do a lot of operations such as moving cursor, visual selection, switching to other buffers and windows, etc. But modifying the buffer is not allowed, i.e. user cannot edit/delete the buffer when it is loading/saving the file on the file system. This is mostly to ensure user data security.
2. If a buffer is associated with a non-existing file and modified, it is always _**changed**_ and will never go to _**synced**_ unless it is been saved on filesystem.
3. All the status are for one certain buffer, there is no other buffer instances in the flow chart.
4. There are still big gaps between the buffer operations and the (user facing) Vim ex commands, i.e. `:edit`, `:file`, etc. This is an internal middle-level design.

### Data Racing

Before showing the data racing examples, let's introduce one more buffer API to fetch the buffer's info and data:

```typescript
// Buffer status enums.
enum BufferStatus {
    Initialized = 1,
    Loading,
    Saving,
    Synced,
    Changed,
}

// Buffer metadata.
type BufferMetadata {
    // Buffer status.
    status: BufferStatus;

    // Associated file name of the buffer.
    // For detached buffer, this field is `undefined`.
    fileName?: string;

    // Buffer contents.
    contents?: string;

    // Last modified time of the file (on the filesystem).
    // For detached buffer, this field is `undefined` since it's never synced with filesystem.
    lastModifiedTime?: Date;

    // Create time of the file (on the filesystem).
    // For detached buffer, this field is `undefined` since it's never synced with filesystem.
    createTime?: Date;

    // Last synced time between buffer and filesystem.
    // For detached buffer, this field is `undefined` since it's never synced with filesystem.
    lastSyncedTime?: string;
}

// Fetch buffer metadata.
// It returns the metadata of a buffer.
Rsvim.buf.getMetadata(bufferId: number): BufferMetadata?;
```

When in an async and multiple-threading tokio runtime, we would want to keep the buffer's data thread-safe and the API behaves like atomic primitives, just like locked by mutex or follows the ACID (atomicity, consistency, isolation, durability) principles in database transactions.

But the `loading` and `saving` status are not easy to handle, image we have below typescripts:

```typescript
// User just type `:edit file1` command to open a file with a new buffer.
let bufferIds = Rsvim.buf.listBuffers();
bufferIds.forEach((bufferId) => {
  console.log(
    `For buffer ${bufferId}, the status is: ${Rsvim.buf.getMetadata(bufferId)?.status}`,
  );
  console.log(`The contents is: ${Rsvim.buf.getMetadata(bufferId)?.contents}`);
});
```

If the above example code runs right after user opens a buffer with `:edit {file1}` ex command, i.e. the editor just runs the `Rsvim.buf.editFile` API in an async task spawned by tokio runtime. In this time, the result of the `Rsvim.buf.getMetadata` API in the example code is highly possible to be `Loading` status, and incomplete/partial contents different from the filesystem.

What should the javascripts do now?

Solution-1: It would use a blocking while-loop to wait for the buffer syncing (loading/saving) complete:

```typescript
// Use while-loop to wait for buffer syncing complete.
while (
  buffersId
    .map((bufferId) => Rsvim.buf.getMetadata(bufferId)?.status)
    .some(
      (status) =>
        status === BufferStatus.Loading || status === BufferStatus.Saving,
    )
) {
  /* Do nothing. */
}

// Syncing complete, continue user logic.
```

Solution-2: The editor have to change all the buffer related APIs to `async`, so let user wait for them syncing complete:

```typescript
// Change the `Rsvim.buf.listBuffers` API to async mode.
// It returns a promise of buffer IDs.
Rsvim.buf.listBuffers(opts?:{includeUnlist:boolean?}): Promise<Array<number>>;

// Then we can list all buffer IDs like this:
let bufferIds = await Rsvim.buf.listBuffers();
// ...
```

The solution-1 of course leads to low performance and bad practice, while solution-2 looks it will need a full-featured buffer-level lock/notify/condition mechanism (similar to mutex, conditional variable and notify in C/C++ pthread or rust sync library), or transactions that support ACID principles in relational database.

The conclusion is: we will fall into the data racing trap and never solve it correctly.

## Sync Way

Once we simply use sync IO to synchronize file contents between buffer and filesystem, there will be no `loading` or `saving` status, and APIs exposed to javascript side will be all sync and blocking. The internal dataflow becomes:

![2](../images/4-WindowsAndBuffers-1-FileBuffer.2.drawio.svg)

Actually we no longer need these status, the buffer's metadata itself already indicates the status:

- For associated or detached, a buffer can indicate by whether it has the `fileName`, `lastModifiedTime`, `createTime` and `lastModifiedTime` fields. A detached buffer don't have these fields.
- For buffer status `Initialized`, a buffer can indicate by whether it has no `fileName` and not been modified.
- For buffer status `Synced` and `Changed`, a buffer can indicate by whether it the `fileName`, and also it needs to maintain all the modifications history since the buffer is created.

This is the final easy and stable solution for buffer and filesystem.

## References

- \[1\]: <https://vimhelp.org/editing.txt.html>
- \[2\]: <https://vimhelp.org/editing.txt.html#%3Aedit>
- \[3\]: <https://vimhelp.org/editing.txt.html#%3Aenew>
- \[4\]: <https://vimhelp.org/editing.txt.html#%3Afile_f>
- \[5\]: <https://vimhelp.org/editing.txt.html#%3Asav>
- \[6\]: <https://vimhelp.org/cmdline.txt.html#%3A%2C>
- \[7\]: <https://vimhelp.org/editing.txt.html#%3Awrite_f>
