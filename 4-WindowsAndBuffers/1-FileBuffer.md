# File Buffer

> Written by @linrongbin16, first created at 2024-11-20.

This RFC describes the file buffer that maps the file content from hardware disk to the memory inside editor.

The file buffer has several functionalities:

1. It can be either associated with a file on the file system, or detached with no file.

   > NOTE: The _**file system**_ can be not only a local HDD/SSD disk, but also remote via network, as long as the operating system manages it well. But we would not explicitly handles the network (TCP, UDP, etc) connecting or data transferring.
