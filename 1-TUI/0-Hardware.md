# Hardware

## Spec List

Here's a list of hardwares and specifications for some very popular PC/laptop computers:

### Year 2022

|         | DIY PC                                                                                                 | MacBook Pro                                                                             |
| ------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| CPU     | Intel I5-12400<br/>6 P-Cores, Base Frequency 2.5GHz                                                    | Apple M2 Chip<br/>4 P-Cores, Base Frequency 3.5GHz<br/>4 E-Cores, Base Frequency 2.8GHz |
| GPU     | Intel UHD Graphics 730<br/>Base Frequency 300MHz, Max Frequency 1.45GHz                                | Apple M2 10-Cores GPU<br/>Base Frequency 0.44GHz, Max Frequency 1.4GHz                  |
| Memory  | Kingston 8GB DDR4<br/>2666MT/s                                                                         | 16GB LPDDR5<br/>6400MHz                                                                 |
| Storage | Samsung 512GB PCIe4.0 SSD<br/>Sequential Read/Write 4000-5000 MB/s<br/>Random Read/Write 500-900K IOPS | 512GB SSD<br/>3200MHz                                                                   |

### Year 2014

|         | DIY PC                                                                                          | MacBook Pro                                                         |
| ------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| CPU     | Intel I5-4690K<br/>4 Cores, Base Frequency 3.5GHz                                               | Intel I5 Chip<br/>Dual Cores, Base Frequency 2.6GHz                 |
| GPU     | HD Graphics 4600<br/>Base Frequency 350MHz, Max Frequency 1.2GHz                                | Intel Iris Graphics<br/>Base Frequency 200MHz, Max Frequency 1.2GHz |
| Memory  | Kingston 8GB DDR4<br/>1666MT/s                                                                  | 8GB DDR3L<br/>1600MHz                                               |
| Storage | Western Digital 512GB HDD<br/>Sequential Read/Write 150-200 MB/s<br/>Random Read/Write 1-5 MB/s | 512GB Flash Storage<br/>Read/Write 700-1000 MB/s                    |

## Conclusion

A very basic concept for the hardware speed from highest to lowest is: CPU > Memory > Max Freuency GPU > IO Devices > Base Frequency GPU. For a PC/laptop, the terminal printing is related to kernel, desktop GUI and GPU, so roughly speaking the speed is even or slower than IO devices.

This is also the basic rules when rendering the terminal, for the same amount of data:

- CPU calculating/processing is preferred over flushing to terminal.
- Sequential flushing to terminal is preferred over random flushing.
