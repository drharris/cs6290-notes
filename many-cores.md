---
id: many-cores
title: Many Cores
sidebar_label: Many Cores
---

[🔗Lecture on Udacity (45 min)](https://classroom.udacity.com/courses/ud007/lessons/913459012/concepts/last-viewed)

## Many-Core Challenges
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Coherence Traffic \\(\uparrow\\)
  - Writes to shared location \\(\Rightarrow\\) invalidations and misses
  - Cores \\(\uparrow\\) \\(\Rightarrow\\) more writes/sec \\(\Rightarrow\\) bus throughput \\(\uparrow\\) 
  - Bus: one request at a time \\(\Rightarrow\\) bottleneck
  - We need:
    - Scalable on-chip network that allows traffic to grow with number of cores
    - Directory coherence (so we don't rely on bus to serialize everything)

## Network On Chip
Consider a linear bus - as we add more cores, it increases the length of the bus (and thus may be slower), and there is also more traffic. It gets very bad with more cores.

Now consider if the cores were arranged in a mesh network, where each core has some adjacent cores it communicates to. Any core can still communicate to any other core through this network, but potentially there can be many such communications going on at once, so the total throughput is much higher, several times what an individual link's throughput.

As we continue to add cores, it simply increases the size of the mesh. While the amount of total traffic increases, so does the total number of links and thus the total throughput of the network.
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Links \\(\uparrow\\) \\(\Rightarrow\\) Available Throughput \\(\uparrow\\)
- This is also good for chip building because the links don't intersect each other - it is somewhat flat and easily built in silicon

Can build various network topologies:
  - Bus - single link connected to all cores
  - Mesh - each core is connected to adjacent cores
  - Torus - A mesh where "ends" are also linked to each other in both horizontal/vertical directions
  - More advanced or less "flat": Flattened Butterfly, Hypercube, etc.

## Many-Core Challenges 2
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Coherence Traffic \\(\uparrow\\)
  - Scalable on-chip network (e.g. a mesh)
  - Directory coherence
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Off-Chip Traffic \\(\uparrow\\)
  - \# cores \\(\uparrow\\) \\(\Rightarrow\\) \# of on-chip caches \\(\uparrow\\)
  - Misses/core same \\(\Rightarrow\\) cores \\(\uparrow\\) \\(\Rightarrow\\) mem requests \\(\uparrow\\)
  - \# of pins \\(\uparrow\\), but not \\(\approx\\) to \# of cores \\(\Rightarrow\\) bottleneck
  - Need to reduce # of memory requests/core
    - Last level cache (L3) shared, size \\(\uparrow \approx \\) # cores \\(\uparrow\\)
    - One big LLC \\(\Rightarrow\\) SLOW ... one "entry point" \\(\Rightarrow\\) Bottleneck (Bad)
    - Distributed LLC

## Distributed LLC
- Logically a single cache (block is not replicated on each cache)
- But sliced up so each tile (core + local caches) gets part of it
  - L3 size = # cores * L3 slice size
  - On an L2 miss, we must request block from correct L3 slice. How do we know which slice to ask?
    - Round-robin by cache index (last few bits of set number)
      - May not be good for locality
    - Round-robin by page #
      - OS can map pages to make accesses more local

## Many-Core Challenges 2 (again)
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Coherence Traffic \\(\uparrow\\)
  - Scalable on-chip network (e.g. a mesh)
  - Directory coherence
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Off-Chip Traffic \\(\uparrow\\)
  - Large Shared Distributed LLC
- Coherence Directory too large (to fit on chip)
  - Entry for each memory block
  - Memory many GB \\(\Rightarrow\\) billions of entries? \\(\Rightarrow\\) can't fit

## On-Chip Directory
- Home node? Same as LLC slice! (we'll be looking at that node anyway)
- Entry for every memory block? No.
- Partial directory
  - Directory has limited # of entries
  - Allocate entry only for blocks that have at least 1 presence bit set (only blocks that might be in at least one of the private caches)
    - If it's only in the LLC or memory, we don't need a directory entry (it would be all zeroes anyway)
  - Run out of directory entries?
    - Pick an entry to replace (LRU), say entry E
    - Invalidation to all tiles with Presence bit set
    - Remove entry E and put new entry there
    - This is a new type of cache miss, caused by directory replacement

## Many-Core Challenges 3
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Coherence Traffic \\(\uparrow\\)
  - Scalable on-chip network (e.g. a mesh)
  - Directory coherence
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Off-Chip Traffic \\(\uparrow\\)
  - Large Shared Distributed LLC
- Coherence Directory too large (to fit on chip)
  - Distributed Partial Directory
- Power budget split among cores
  - Cores \\(\uparrow\\) \\(\Rightarrow\\) W/core \\(\downarrow\\) \\(\Rightarrow\\) f and V \\(\downarrow\\) \\(\Rightarrow\\) 1 thread program is slower with more cores!

## Multi-Core Power and Performance
![multi core power and performance](https://i.imgur.com/cfb3TXo.png)

Due to need to compensate for less power, each core is noticeably slower.

## No Parallelism \\(\Rightarrow\\) boost frequency
- "Turbo" clocks when running on one core
- Example: Intel's Core I7-4702MQ (Q2 2013)
  - Design Power: 37W
  - 4 cores, "Normal" clock 2.2GHz
  - "Turbo" clock 3.2GHz (1.45x normal \\(\Rightarrow\\) 3x power)
    - Why not 4x? It spreads more heat to other cores - 3x keeps it distributed to match normal 2.2GHz
- Example: Intel's Core I7-4771 (Q3 2013)
  - Design Power: 84W
  - 4 cores, "Normal" clock 3.5GHz
  - "Turbo" clock 3.9GHz (1.11x normal \\(\Rightarrow\\) 1.38x power)
    - Meant for desktop, so can cool the chip more effectively at high power
    - But this means the chip already runs almost as hot as it can get, so we don't have much more room to increase power further

## Many-Core Challenges 4
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Coherence Traffic \\(\uparrow\\)
  - Scalable on-chip network (e.g. a mesh)
  - Directory coherence
- Cores \\(\uparrow\\) \\(\Rightarrow\\) Off-Chip Traffic \\(\uparrow\\)
  - Large Shared Distributed LLC
- Coherence Directory too large (to fit on chip)
  - Distributed Partial Directory
- Power budget split among cores
  - "Turbo" when using only one core
- OS Confusion
  - Multi-threading, cores, chips - all at same time!

## SMT, Cores, Chips, ...
All combined:
- Dual socket motherboard (two chips)
- 4 cores on each chip
- Each core 2-way SMT
  - 16 threads can run

What if we run 3 threads?
- Assume OS assigns them to the first 3 spots, but maybe two of those are actually the same core (because SMT), and the first half of those spots are actually on the same chip.
- A smarter policy would be to put them on separate chips if possible, and then separate cores if possible, to maximize all benefits.
- So, the OS needs to be very aware of what hardware is available, and smart enough to use it effectively.


*[LLC]: Last-Level Cache
*[SMT]: Simultaneous Multi-Threading