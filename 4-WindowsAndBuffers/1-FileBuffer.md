# File Buffer

> Written by @linrongbin16, first created at 2024-11-20.

This RFC describes the file buffer that maps the file content from hardware disk to the memory inside editor.

The file buffer has several functionalities:

1. It can be either associated with a file on the file system, or detached with no file.

   > NOTE: The _**file system**_ can be not only a local HDD/SSD disk, but also remote via network, as long as the operating system manages it well. But we would not explicitly handles the network (TCP, UDP, etc) connecting or data transferring.

2. It has several buffer status, a certain buffer status in a certain time point:

   - Initialized: When a file buffer is created, it is always detached, and the status is always `initialized`.
   - Loading: When a file buffer is first binded to (i.e. becomes associated with) a file, the status changes to `loading`, i.e. the editor reads (no matter in sync or async) the file content into buffer.
   - Loaded: When the reading operation is completed, the status changes to `loaded`.
   - Changed: When user edits the buffer, the status changes to `changed`, i.e. the buffer's content is different from disk file's content.
   - Saving: When the reading operation is completed, the status changes to loaded.

3. The file buffer's two states (associated and detached) can be switched to each other, in both sync/async manner, with some restrictions:
