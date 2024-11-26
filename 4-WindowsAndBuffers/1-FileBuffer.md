# File Buffer

> Written by @linrongbin16, first created at 2024-11-21.

This RFC describes the file buffer that maps the file content from hardware disk to the memory inside editor.

The file buffer has several functionalities:

1. It can be either associated with a file on the file system, or detached with no file.

   > NOTE: The _**file system**_ can be not only a local disk storage, but also a remote file system via network protocols.

2. It has several status, a certain time point has a certain status:

   - Initialized: When a file buffer is created, it is always detached, and the status is always _**initialized**_.
   - Changed: When buffer contents are different from file system (if associated) or once been modified (if detached). Note: once a detached buffer is been modified, it will always stay in _**changed**_ status, and cannot go back to _**initialized**_ status.
   - Loading: When the buffer reads the content from associated file into the buffer, the status is _**loading**_. Note: a detached buffer cannot change its status to _**loading**_.
   - Saving: When the buffer writes the buffer's contents into the associated file, the status is _**saving**_. Note: a detached buffer cannot change its status to _**saving**_.
   - Synced: When the buffer's loading/saving operation is completed, the status changes to _**synced**_.

The two states (associated and detached) and several running status can be switched to each other:

![1](../images/4-WindowsAndBuffers-1-FileBuffer.1.drawio.svg)

Note:

1. `Loading` and `Saving` are momentary states, since Rsvim is designed with the concept of never freezing, file operations are handled with async manner and won't block TUI. During these states, user can still do a lot of operations such as moving cursor, switching to other buffers, etc. But buffer modification is not allowed, i.e. user cannot edit/delete the buffer when it is reading/writing contents from/to the file on the file system. This is mostly to ensure user data security.
2. The `bind to` is a general word to describe how the buffer becomes associated with a file, when implementing it in Rsvim editor, it can be some APIs that can let users running the `:edit` or `:file` vim ex command.
3. If a buffer is detached and modified, it is always _**changed**_ and will never go back to _**initialized**_.
4. If a buffer is associated with a non-existing file and modified, it is always _**changed**_ and will never go to _**synced**_ unless it is been saved.
