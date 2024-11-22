# File Buffer

> Written by @linrongbin16, first created at 2024-11-21.

This RFC describes the file buffer that maps the file content from hardware disk to the memory inside editor.

The file buffer has several functionalities:

1. It can be either associated with a file on the file system, or detached with no file.

   > NOTE: The _**file system**_ can be not only a local disk storage, but also a remote file system via network, as long as the operating system manages it well. But we would not explicitly handles the network (TCP, UDP, etc) connecting or data transferring.

2. It has several status, a certain time point has a certain status:

   - Initialized: When a file buffer is created, it is always detached, and the status is always _**initialized**_.
   - Loading: When the editor reads the content from associated file into the buffer, the status is _**loading**_.
   - Loaded: When the loading (reading operation) is completed, the status changes to _**loaded**_. Note: a detached buffer has to first bind to a file on the file system before it loads the file content, i.e. it becomes associated with a file.
   - Changed: When user edits the buffer, the status changes to _**changed**_, i.e. the buffer's content is different from the file on the file system. While for the detached buffer, once the content is not empty it is always _**changed**_ unless it is been saved into a file on the file system (and it becomes associated with that file).
   - Saving: When the editor writes the buffer's contents into the associated file, the status is _**saving**_. Note: a detached buffer has to first bind to a file on the file system before it saves its content into the file, i.e. it becomes associated with a file.

The two states (associated and detached) and several running status can be switched to each other:

![1](../images/4-WindowsAndBuffers-1-FileBuffer.1.drawio.svg)

Note:

1. `Loading` and `Saving` are momentary states, since Rsvim is designed with the concept of never freezing, file operations are handled with async manner and won't block TUI. But during these states, buffer modifications are still not allowed, i.e. user cannot edit/delete buffers when they are reading/writing contents from/to the file on the file system, unless the editor process is been force killed.
