---
id: memory
title: Memory
sidebar_label: Memory
---

[🔗Lecture on Udacity (23 min)](https://classroom.udacity.com/courses/ud007/lessons/872590120/concepts/last-viewed)

## How Memory Works
* Memory Technology: SRAM and DRAM
* Why is memory slow?
* Why don't we use cache-like memory?
* What happens when we access main memory?
* How do we make it faster?

## Memory Technology SRAM and DRAM
SRAM - Static Random Access Memory - Faster
- Retains data while power supplied (good)
- Several transistors per bit (bad)

DRAM - Dynamic Random Access Memory - Slower
- Will lose data if we don't refresh it (bad)
- Only one transistor per bit (good)

Both of these lose data when power is not supplied

## One Memory Bit
### SRAM
* [🎥 See SRAM (4:13)](https://www.youtube.com/watch?v=mwNqzc1o5zM)

SRAM is a matrix of Wordlines and Bitlines. The wordline controls which word is selected, which is the "on" switch for transistors for that word. These transistors connect the bitlines to a inverter feedback loop that contains the data. Another transistor connects the other side of the inverter to another bitline that represents the inverted value of that same bit. In this way memory writes can more easily happen by driving the inverter loop in the opposite direction at the same times, and we can also be more sure of the bit value by looking at the difference between the bit and not-bit lines.

### DRAM
* [🎥 See DRAM (4:03)](https://www.youtube.com/watch?v=3s7zsLU83bY)

In DRAM we similarly have a transistor controlled by the wordline, but the main difference is that our memory is no longer stored in a feedback loop, but in a simple capacitor that is charged on a write to memory. The problem is that a transistor is not a perfect switch - it's a tiny bit leaky, which means the capacitor will lose its charge over time. So, we need to write that bit again from time to time. Additionally, reads are destructive, in that they also drain the capacitor.

### SRAM/DRAM Thoughts

We also see that SRAM can be called a 6T cell (2 control transistors + two transistors each inverter). DRAM is a 1T cell (single transistor controlling the capacitor). We might think the physical footprint of that is a single transistor plus a capacitor (and we want a large capacitor to retain the value longer). However, DRAM uses a technology called "trench cell", which buries a capacitor underneath transistor into a single unit as far as footprint. Additionally, we don't need the second bitline as in SRAM, so space is conserved and the cost is much lower overall.

## Memory Chip Organization
A Row Address is passed into a Row Decoder, which decides which Wordline to activate. There is a memory cell on each intersection between a wordline and a bitline. Once a row is activated, the bits in that word follows down the bitlines at a small voltage where it is collected in a Sense Amplifier that produces full 0 and 1 bits. These signals go into a Row Buffer, a storage element for these values. The value is then fed into a Column Decoder, which takes a Column Address (which bit to read), and outputs a single bit. This can then be replicated to provide more bits.
![Memory Chip Organization](https://i.imgur.com/ABBEPWh.png)

After this happens, the destructive read means the value needs to be refreshed. So the sense amplifier now drives the bitlines to ensure the value has been refreshed. Thus, every read is really a Read-Then-Write. It also must handle refreshing all words periodically to counteract leakage. We can't rely on the processor to do this for us, because things that are accessed often are probably in cache and won't be accessed in memory. So, we have a Refresh Row Counter that loops through words within the required refresh period. So for a Refresh Period T and N rows, then every T/N seconds we need to refresh a row. In DRAM there are many rows and T is less than a second, so this is a lot of required refresh activity.

To write to memory, we still read a row, and once it is latched into the row buffer we provide the new value to the column decoder, which then will write it into the row buffer, and the sense amplifier uses the value to write back on the bitlines. Thus, a write is also a Read-Then-Write, similar to the read.

## Fast Page Mode
"Page" is the series of bitlines that can be read by selecting the proper row. It can be many bits long. Fast Page mode is a technique in which once a row is selected, the value is latched into the row buffer and successive bit reads will happen faster.
- Fast Page Mode
  - Open a page
    - Row Address 
    - Select Row
    - Sense Amplifier
    - Latch into Row Buffer
  - Read/Write Cycle (can be many read and write operations on that page)
  - Close the page
    - Write data from row buffer back to memory row

## Connecting DRAM to the Processor
Once a memory request has missed all levels of cache, it is communicated externally through the Front Side Bus into the Memory Controller. The Memory Controller channels these requests into one of several DRAM channels. Thus, the total memory latency includes all the overhead of the communication over the FSB, the activities of the Memory Controller, in addition to the memory access itself.
![Connecting DRAM](https://i.imgur.com/MNIeFRR.png)

Some recent processors integrate the memory controller so we can eliminate communication over the FSB and communicate directly from it. This can be a significant overall savings. It also forced standardzation of the protocols connecting DRAM to the processor to allow for this without a complete chip redesign.



*[DRAM]: Dynamic Random Access Memory
*[SRAM]: Static Random Access Memory