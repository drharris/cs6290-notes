---
id: cache-review
title: Cache Review
sidebar_label: Cache Review
---

[🔗Lecture on Udacity (1.5 hr)](https://classroom.udacity.com/courses/ud007/lessons/1025869122/concepts/last-viewed)

## Locality Principle
Things that will happen soon are likely to be close to things that just happened.

### Memory References
Accessed Address `x` Recently
- Likely to access `x` again soon (Temporal Locality)
- Likely to access addresses close to `x` too (Spatial Locality)

### Locality and Data Accesses
Consider the example of a library (large information but slow to access). Accessing a book (e.g. Computer Architecture) has Temporal Locality (likely to look up the same information again) and also Spatial Locality (likely to look up other books on Computer Architecture.) In this example, consider options a student can do:
1. Go to the library every time to find the info he needs, go back home
2. Borrow the book and bring it home
3. Take all the books and build a library at home

Option 1 does not benefit from locality (inconvenient). Option 3 is still slow to access and requires a lot of space and does not benefit from locality. Option 2 is a good tradeoff for being able to find information quickly without being overwhelmed with data. This is like a Cache, where we bring only certain information to the processor for faster access.

## Cache Lookups
We need to it be fast, so it must be small. Not everything will fit.
Access
- Cache Hit - found it in the cache (fast access)
- Cache Miss - not in the cache (access slow memory)
  - Copy this location to the cache

## Cache Performance
Properties of a good cache:
- Average Memory Access Time (AMAT)
  - \\(\text{AMAT} = \text{hit time} + \text{miss rate} * \text{miss penalty} \\)
  - hit time \\(\Rightarrow\\) small and fast cache
  - miss rate \\(\Rightarrow\\) large and/or smart cache
  - miss penalty \\(\Rightarrow\\) main memory access time
  - "miss time" = hit time + miss penalty
  - Alternate way to see it: \\(AMAT = (1-rate_{miss}) * t_{hit} + rate_{miss}*t_{miss} \\)

## Cache Size in Real Processors
Complication: several caches

L1 Cache - Directly service read/write requests from the processor
- 16KB - 64KB
- Large enough to get \\(\approx\\) 90% hit rate
- Small enough to hit in 1-3 cycles

## Cache Organization
![Cache Organization](https://i.imgur.com/PIWX0aG.png)

Cache is a table indexed by some bits correlating to the address. It contains data and a flag whether it's a hit or not. The size of data per entry (block size) is selected as a balance between having enough data to maximize hit rate for locality, without using up too much data that will not be accesses. Typically 32-128 bytes is ideal.

### Cache Block Start Address
- Anywhere? 64B block => 0..63, 1..64, 2..65. (cannot effectively index into cache, contains overlap)
- Aligned! 64B block => 0..63, 64..127, ... (easy to index using bits of the address, no overlap)

### Blocks in Cache and Memory
Memory is divided into blocks. Cache is divided into lines (of block size). This line is a space/slot in which a block can be placed.

### Block Offset and Block Number
Block Offset is the lower N bits (where 2^N = block size) that tell us where in the block to index. Block Number is the upper M bits (where M+N = address length) that tell us how to index the blocks for selection.

### Cache Tags
In addition to the data, the cache keeps a list of "tags", one for each cache line. When accessing a block, we compare block number to each tag in the cache to determine which line that block is in, if it is in cache. We then know the line and offset to access the memory.

### Valid Bit
What happens if the tags are initialized to some value that matches the tag of a valid address? We also need a "valid bit" for each cache line to tell us that specific cache line is valid and can be properly read. An added benefit is that we don't need to worry about clearing tag and data all the time - we can just clear the valid bit.

## Types of Caches
- Fully Associative: Any block can be in any line
- Set-Associative: N lines where a block can be
- Direct-Mapped: A particular block can go into 1 line (Set-associative with N==1)

### Direct-Mapped Cache
Each block number maps to a single potential cache line where it can be. Typically some lowermost bits of the block number are used to index into the cache line. The tag is still needed to tell us which block is actually in the cache line, but we now only need the bits of the tag that were not mapped for the index.
![Direct-Mapped Cache](https://i.imgur.com/GlCyPW2.png)

#### Upside and Downside of Direct-Mapped Cache
- Look in only one place (fast, cheap, energy-efficient)
- Block must go in one place ()
  - Conflicts when two blocks are used that map to the same cache line - increased cache miss rate

### Set-Associative Caches
N-Way Set-Associative: a block can be in one of N lines (each set has N lines, and a particular block can be in any line of that set)

#### Offset, Index, Tag for Set-Associative
Similar to before - lower offset bits in the address still determine where in the cache line to obtain the data. Index bits (determined by how many sets) now are used to determine which set the line may be in. The tag is the remaining upper bits bits.
![](https://i.imgur.com/3ehF4Ey.png)

### Fully-Associative Cache
Still have lower offset bits, but there is now no index. All remaining bits are the tag. Any block can be in any cache line, but this means to find something in cache you have to look at every line to see if that is it.

### Direct-Mapped and Fully Associative 
Direct-Mapped = 1-way set associative

Fully Associative = N-way set associative

Always start with offset bits, based on block size. Then, determine index bits. Rest of the bits is the tag.

- Offset bits = \\(log_2(\text{block size})\\)
- Index bits = \\(log_2(\text{# of sets})) \\)

## Cache Replacement
- Set is full
- Miss -> Need to put new block in set
- Which block do we kick out?
  - Random
  - FIFO
  - LRU (Least Recently Used)
    - Hard to implement but exploits locality
    - Could implement via NMRU (Not Most Recently Used)

### Implementing LRU
LRU has a separate set of counters (one per line). The counter is set to max when the line is accessed, and all other counters are decremented. A counter of 0 value represents least recently used and that line could be replaced.

For an N-way SA cache:
- Cost: N log2(N)-bit counters
- Energy: Change N counters on every access
- (Expensive in both hardware and energy)

[🎥 Link to lecture (5:11)](https://www.youtube.com/watch?v=bq6N7Ym81iI)

## Write Policy
Do we insert into the cache blocks we write?
- Write-Allocate
  - Most are write-allocate due to locality - if we write something we are likely to access it
- No-Write-Allocate

Do we write just to cache or also to memory?
- Write-Through (update mem immediately)
  - Very unpopular
- Write-Back (write to cache, but write to mem when block is replaced)
  - Takes advantage of locality, more desirable

If you have a write-back cache you also want write-allocate (with a write miss, we want future writes to go to the cache).

### Write-Back Cache
If we're replacing a block in cache, do we know we need to write it to memory?
- If we did write to the block, write to memory
- If we did not write to the block, no need to write
- Use a "Dirty Bit" to determine if the block has been written to.
  - 0 = "clean" (not written since last brought from memory)
  - 1 = "dirty" (need to write back on replacement)

[🎥 Link to example (3:19)](https://www.youtube.com/watch?v=xU0ICkgTLTo)

## Cache Summary
[🎥 First Part (2:32)](https://www.youtube.com/watch?v=MWpy5bBxl5A)

[🎥 Second Part (2:32)](https://www.youtube.com/watch?v=DhxAIKaCEBY)