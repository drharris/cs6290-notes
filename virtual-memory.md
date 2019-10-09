---
id: virtual-memory
title: Virtual Memory
sidebar_label: Virtual Memory
---

[🔗Lecture on Udacity (1 hr)](https://classroom.udacity.com/courses/ud007/lessons/1032798942/concepts/last-viewed)

## Why Virtual Memory?

Virtual memory is a way of reconciling how a programmer views memory and how the hardware actually uses memory. Every program uses the same address space, even though clearly the data in some of those addresses must be different for each program (e.g. code section).

![why virtual memory](https://i.imgur.com/CoxI4GQ.png)

## Processor's View of Memory

Physical Memory - *Actual* memory modules in the system.
- Sometimes < 4GB
- Almost never 4GB/process
- Never 16 Exabytes/process
- So... usually less than what programs can access

Addresses
- 1:1 mapping to bytes/words in physical memory (a given address always goes to the same physical location, and the same physical location always has the same address)

## Program's View of Memory
Programs each have their own address space, some of which may point to the same physical location as another program's, or not. Likewise, even different addresses in different programs could be pointing at the same shared physical location.

![program's view of memory](https://i.imgur.com/GvJRe0z.png)

## Mapping Virtual -> Physical Memory

Virtual Memory is divided into pages of some size (e.g. 4kB). Likewise, Physical Memory is divided into Frames of that same size. This is similar to Blocks/Lines in caches. The Operating System decides how to map these pages to frames and handles any shared memory using Page Tables, each of which is unique to the program being mapped.

![mapping virtual to physical memory](https://i.imgur.com/0yOtU4H.png)

## Where is the missing memory?

What happens when we have more pages in our programs than can be currently loaded into physical memory? Since virtual memory will almost certainly exceed the size of physical memory, this occurs often. The Operating System will "page" memory to disk that has not been accessed recently, to allow the more used pages to remain in physical memory. Similar to caches, it will then obtain those pages back from disk whenever they are requested.

## Virtual to Physical Translation
Similar to a cache, the lower N bits of a virtual address (where 2^N = page size) are used as an offset into the page. The upper bits are then the page number (similar to tags in caches), which is used to index into the Page Table. This table returns a physical Frame Number, which is then combined with the Page Offset to get the Physical Address actually used in hardware.

## Size of Flat Page Table
- 1 entry per page in the virtual address space
  - even for pages the program never uses
- Entry contains Frame Number + bits that tell us if the page is accessible
  - Entry Size \\(\approx\\) physical address
- Overall Size: \\(\frac{\text{virtual memory}}{\text{page size}}*\text{size of entry}\\)
- For very large virtual address space, the page table may need to be reorganized to ensure it properly fits within memory

## Multi-Level Page Tables
Multi-Level Page Tables are designed where page table space is proportional to how much memory the program is actually using. As most programs use the "early" addresses (code, data, heap) and "late" addresses (stack), there is a large gap in the middle that usually goes unused.
![multi-level page tables](https://i.imgur.com/gQSVDjS.png)

### Multi-Level Page Table Structure
The page number now is split into "inner" and "outer" page numbers. The Inner Page Number is used to index into an Inner Page Table to find the frame number, and the Outer Page Number is used to determine which Inner Page Table to use.
![multi-level page table structure](https://i.imgur.com/XJ0lmoO.png)

### Two-Level Page Table Example
We save space because we do not create an inner page table if nothing is using the associated index in the outer page table. Only a certain number of outer page numbers will be utilized, and they are likely contiguous, so this results in a lot of savings.
![two-level page table example](https://i.imgur.com/weeBBdr.png)

### Two-Level Page Table Size
To calculate two-level page table size:
1. Determine how many bits are used for the page offsets (N bits where \\(2^N\\)=page size)
2. Determine how many bits are used for the inner and outer page tables (X and Y bits where \\(2^X\\)=outer page table entries and \\(2^Y\\)=inner page table entries)
3. Outer Page Table size is number of outer entries * entry size in bytes
4. Inner Page Table size is number of inner entries * entry size in bytes
5. Number of Inner Page Tables Used is determined by partitioning the utilized address space by the number of bits associated with the outer page table. Typically only a few outer entries will be used.
6. Total page table size = (outer page table size) + (number of inner page tables used)*(inner page table size)

[🎥 Link to example (4:19)](https://www.youtube.com/watch?v=tP7LYbFrk10)

## Choosing the Page Size

Smaller Pages -> Large Page Table

Larger Pages -> Smaller Page Table
- But, internal fragmentation due to most of a page not being used by applications (wasted in physical memory)

Like with block size of caches, we need to compromise between these. Typically a few KB to a few MB is appropriate for page size.

## Memory Access Time with V->P Translation
Example: `LOAD R1=4(R2)` (virtual address of value in `R2`+4)

For Load/Store
- Compute Virtual Address
- Compute page number (take some bits from address)
- When using virtual->physical translation (for each level of page table)
  - Compute physical address of page table entry (adding)
  - Read page table entry
    - Is it fast? Where is the page table? In memory!
  - Compute physical address
- Access Cache (and sometimes memory)

Since page table is in memory, this could also result in cache misses, and so it may take multiple rounds of memory access just to get the data in one address.

## Translation Look-Aside Buffer (TLB)
- TLB is a cache for translations
- Cache is big -> TLB is small
  - 16KB in cache is only 4 entries (page size of 4KB)
  - So, TLB can be very small and fast since it only holds translations
- Cache accessed for each level of Page Table (4-level = 4 accesses)
  - TLB only stores the final translation (Frame Number) (1 access)

What if we have a TLB miss?
- Perform translation using page table(s)
- Put translation in TLB for later use

## TLB Organization
- Associativity? Small, fast => Fully or Highly Associative
- Size? Want hits similar to cache => 64..512 entries
  - Need more? Two-level TLB
    - L1: small/fast
    - L2: hit time of several cycles but larger

