# TUI

> Written by @linrongbin16, 2024-08-23

This RFC describes the rendering system inside TUI.

## Background

Here's part lists of hardwares and benchmarks for some very popular PC/laptop computers:

| Year       |         | DIY Windows PC                                                                                      | MacBook Pro                                                                                                            |
| ---------- | ------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Since 2022 | CPU     | Intel I5-12400<br/>6 Cores (6 P-Cores), Base Frequency 2.5GHz, Max Turbo Frequency 4.40GHz          | Apple M2 Silicon Chip<br/>8 Cores (4 P-Cores, 4 E-Cores), P-Cores Base Frequency 3.5GHz, E-Cores Base Frequency 2.8GHz |
|            | Memory  | Kingston 8GB DDR4<br/>2666MT/s                                                                      | 16GB LPDDR5<br/>6400MHz                                                                                                |
|            | Storage | Samsung 512GB PCIe4.0 SSD<br/>Sequential Read/Write 4000-5000 MB/s, Random Read/Write 500-900K IOPS | 512GB SSD<br/>3200MHz                                                                                                  |
| Since 2014 | CPU     | Intel I5-4690K<br/>4 Cores, Base Frequency 3.5GHz, Max Turbo Frequency 3.90GHz                      | Intel I5 Chip<br/>Dual Cores, Base Frequency 2.6GHz, Turbo Boost Frequency 3.1GHz                                      |
|            | Memory  | Kingston 8GB DDR4<br/>1666MT/s                                                                      | 8GB DDR3L<br/>1600MHz                                                                                                  |
|            | Storage | Western Digital 512GB HDD<br/>Sequential Read/Write 150-200 MB/s, Random Read/Write 1-5 MB/s        | 512GB Flash Storage<br/>2.8GHz                                                                                         |

A very basic concept for the hardware speed from highest to lowest is: CPU > Memory > Storage. This is also the basic rule we're following when rendering text contents to terminal:

- For same amount of data, CPU calculating/processing is preferred over flushing to IO devices.
- For same amount of data, sequential flushing to IO devices is preferred over random flushing.

The rendering system splits into 3 layers: UI widget tree, canvas and hardware device. It looks like:

```text

```
