---
id: advanced-caches
title: Advanced Caches
sidebar_label: Advanced Caches
---

[🔗Lecture on Udacity (1.5 hr)](https://classroom.udacity.com/courses/ud007/lessons/1080508587/concepts/last-viewed)

## Improving Cache Performance
Average Memory Access Time: \\(\text{AMAT} = \text{hit time} + \text{miss rate} * \text{miss penalty} \\)
- Reduce hit time
- Reduce miss rate
- Reduce miss penalty

## Reduce Hit Time
- Reduce cache size (bad for miss rate)
- Reduce cache associativity (bad for miss rate)
- Overlap cache hit with another hit
- Overlap cache hit with TLB hit
- Optimize lookup for common case
- Maintain replacement state more quickly

### Pipelined Caches (overlap with another hit)
- Multiple Cycles to access cache
  - Access comes in cycle N (hit)
  - Access comes in cycle N+1 (hit) - has to wait
- Hit time = actual hit + wait time
- Pipeline in 3 stages (for example):
  - Stage 1: Index into set, read out tags and valid bits in that set to determine hit
  - Stage 2: Combining hits to determine did we hit overall and where in our set, begin the read of data
  - Stage 3: Finish data read by using offset to choose the right part of the cache block and returning it

## TLB and Cache Hit
Hit time = TLB access time + Cache access time
![TLB and cache hit](https://i.imgur.com/5yCJxdz.png)
This represents the "Physically Indexed Physically Tagged" cache (PIPT), since both the tags and indexes are physical

## Virtually Accessed Cache
Also called VIVT (Virtual Indexed Virtually Tagged). The virtual address first accesses the cache to get the data, and only goes to TLB to get physical address on a cache miss. Cache is now tagged by virtual address instead of physical. This means hit time is simply the cache hit time. An additional benefit could be no TLB access on a cache hit, but in reality we still need TLB information like permissions (RWX) so we still need to do a TLB lookup even on a cache hit.

A bigger issue is that virtual address is specific to each process - so, virtual addresses that are the same will likely be pointing to different physical locations. This can be solved by flushing the cache on a context switch, which causes a burst of cache misses each time we come back to that process.

## Virtually Indexed Physically Tagged (VIPT)
In this cache, we use the index of the virtual address to get the correct set in cache, and we look at tags in that set. Meanwhile, we also retrieve frame number using the Tag from the virtual address. This frame number is used to compare against the tags in the indexed cache set. The hit time for this cache is the max of cache hit time and TLB lookup time, as these activities occur in parallel. Additionally, we do not have to flush the cache on a context switch, because the cache lines are tagged with physical address.
![VIPT](https://i.imgur.com/IHCm6sK.png)

## Aliasing in Virtually Accessed Caches
Aliasing is a problem with virtually accessed caches because multiple virtual addresses could legitimately point to the same physical location in memory, yet be cached separately due to different indices from the virtual addresses. If one location is modified, a cached version of the other virtual address may not see the change.

![Aliasing in virtually accessed caches](https://i.imgur.com/D7Oeama.png)

## VIPT Cache Aliasing
Depending on how the cache is constructed, it is possible to avoid aliasing issues with VIPT. In this example, the index and offset bits were both part of the frame offset, so since VIPT caches use a physical tag, it will always retrieve the same block of memory from cache even if it is accessed using multiple virtual addresses. But, this requires the cache to be constructed in a way that takes advantage of this - the number of sets must be limited to ensure the index and offset bits stay within the bits used for frame offset
![VIPT Cache Aliasing](https://i.imgur.com/Oq5TWi4.png)

## Real VIPT Caches
\\(\text{Cache Size} \leq \text{Associativity} * \text{Page Size}\\)
- Pentium 4:
  - 4-way SA x 4kB \\(\Rightarrow\\) L1 is 16kB
- Core 2, Nehalem, Sandy Bridge, Haswell:
  - 8-way SA x 4kB \\(\Rightarrow\\) L1 is 32kB
- Skylake (rumored):
  - 16-way SA x ?? \\(\Rightarrow\\) L1 is 64kB

## Associativity and Hit Time
- High Associativity:
  - Fewer conflicts \\(\rightarrow\\) Miss Rate \\(\downarrow\\) (good)
  - Larger VIPT Caches \\(\rightarrow\\) Miss Rate \\(\downarrow\\) (good)
  - Slower hits (bad)
- Direct Mapped:
  - Miss Rate \\(\uparrow\\) (bad)
  - Hit Time \\(\downarrow\\) (good)

Can we cheat on associativity to take the good things from high associativity and also get better hit time from direct mapped?

## Way Prediction
- Set-Associative Cache (miss rate \\(\downarrow\\))
- Guess which line in the set is the most likely to hit (hit time \\(\downarrow\\))
- If no hit there, normal set-associative check

### Way Prediction Performance
| | 32kB, 8-way SA | 4kB Direct Mapped | 32kB, 8-way SA with Way Pred |
|--------------|:---:|:---:|:------:|
| Hit Rate     | 90% | 70% |   90%  | 
| Hit Latency  |  2  |  1  | 1 or 2 |
| Miss Penalty | 20  | 20  |   20   |
| AMAT         |  4<br />(\\(2+0.1\*20\\)) | 7<br />(\\(1+0.3\*20\\)) | 3.3 (1.3+2)<br />(\\(0.7\*1+0.3\*2+0.1\*20\\))

## Replacement Policy and Hit Time
- Random 
  - Nothing to update on cache hit (good)
  - Miss Rate \\(\uparrow\\) (bad)
- LRU
  - Miss Rate \\(\downarrow\\) (good)
  - Update lots of counters on hit (bad)
- We want benefit of LRU miss rate, but we need less activity on a cache hit

## NMRU Replacement 
(Not Most Recently Used)
- Track which block in set is MRU (Most Recently Used)
- On Replacement, pick a non-MRU block

N-Way Set-Associative Tracking of MRU
- 1 MRU pointer/set (instead of N LRU counters)

This is slightly weaker than LRU since we're not ordering the blocks or have no "future" knowledge - but we get some of the benefit of LRU without all the counter overhead.

## PLRU (Pseudo-LRU)
- Keep one bit per line in set, init to 0
- Every time a line is accessed, set the bit to 1
- If we need to replace something, choose a block with bit set to 0
- Eventually all the bits will be set - when this happens, we zero out the other bits.
  - This provides the same benefit as NMRU

Thus, at any point in time, PLRU is at least as good as NMRU, but still not quite as good as true LRU. But, we still get the benefit of not having to perform a lot of activities - just simple bit operations.

## Reducing the Miss Rate
What are the causes of misses?

Three Cs:
- Compulsory Misses
  - First time this block is accessed
  - Would still be a miss in an infinite cache
- Capacity Misses
  - Block evicted because of limited cache size
  - Would be a miss even in a fully-associative cache of that size
- Conflict Misses
  - Block evicted because of limited associativity
  - Would not be a miss in fully-associative cache

## Larger Cache Blocks
- More words brought in on a miss
  - Miss Rate \\(\downarrow\\) when spatial locality is good (good)
  - Miss Rate \\(\uparrow\\) when spatial locality is poor (bad)
  - As block size increases, miss rate decreases for awhile, and then begins to increase again. 
    - The local minima is the optimal block size (64B on a 4kB cache)
    - For larger caches, the minima will occur at a much higher block size (e.g. 256B for 256kB)

## Prefetching
- Guess which blocks will be accessed soon
- Bring them into cache ahead of time
  - This moves the memory access time to before the actual load
- Good guess - eliminate a miss (good)
- Bad guess - "Cache Pollution", and we get a cache miss (bad)
  - We might have also used a cache spot that could have been used for something else, causing an additional cache miss.

### Prefetch Instructions
We can make an instruction that allows the programmer to make explicit prefetches based on their knowledge of the program. In the below example, it shows that this isn't necessarily easy - choosing a wrong "distance" to prefetch is difficult. If you choose it too small, the prefetch is too late and we get no benefit. If it is too large, we prefetch too early and it could be replaced by the time we need it. An additional complication is that you must know something about the hardware cache in order to set this value appropriately, making it not very portable.
![prefetch instructions](https://i.imgur.com/4a0zHOV.png)

### Hardware Prefetching
- No change to program
- HW tries to guess what will be accessed soon
- Examples:
  - Stream Buffer (sequential - fetch next block)
  - Stride Prefetcher (continue fetching fixed distance from each other)
  - Correlating Prefetcher (keeps a history to correlate block access based on previous access)

## Loop Interchange
Replaces the order of nested loops to optimize for bad cache accesses. In the example below it was fetching one element per block. A good compiler should reverse these loops to ensure cache accesses are as sequential as possible. This is not always possible - the compiler has to prove the program is still correct once reversing the loops, which it does by showing there are no dependencies between iterations.
![loop interchange](https://i.imgur.com/JMPw2Fk.png)

## Overlap Misses
Reduces Miss Penalty. A good out of order procesor will continue doing whatever work it can after a cache miss, but eventually it runs out of things to do while waiting on the load. Some caches are blocking, meaning a load must wait for all other loads to complete before starting. This creates a lot of inactive time. Other caches are Non-Blocking, meaning the processor will use Memory-Level Parallelism to overlap multiple loads to minimize the inactive time and keep the processor working.
![overlap misses](https://i.imgur.com/4kJzxCX.png)

### Miss-Under-Miss Support in Caches
- Miss Status Handling Registers (MSHRs)
  - Info about ongoing misses
  - Check MSHRs to see if any match
    - No Match (Miss) \\(\Rightarrow\\) Allocate an MSHR, remember which instruction to wake up
    - Match (Half-Miss) \\(\Rightarrow\\) Add instruction to MSHR
  - When data comes back, wake up all instructions waiting on this data, and release MSHR
- How many MSHRs do we want?
  - 2 is good, 4 is even better, and there are still benefits from level larger amount like 16-32.

## Cache Hierarchies
Reduces Miss Penalty. Multi-Level Caches:
- Miss in L1 cache goes to L2 cache
  - L1 miss penalty \\(\neq\\) memory latency
  - L1 miss penalty = L2 Hit Time + (L2 Miss Rate)*(L2 Miss Penalty)
- Can have L3, L4, etc.

### AMAT With Cache Hierarchies
\\(\text{AMAT} = \text{L1 hit time} + \text{L1 miss rate} * \text{L1 miss penalty} \\)

\\(\text{L1 Miss Penalty} = \text{L2 hit time} + \text{L2 miss rate} * \text{L2 miss penalty} \\)

\\(\text{L2 Miss Penalty} = \text{L3 hit time} + \text{L3 miss rate} * \text{L3 miss penalty} \\)

... etc, until:

\\(\text{LN Miss Penalty} = \text{Main Memory Latency}\\) (Last Level Cache - LLC)

### Multilevel Cache Performance

|          | 16kB | 128kB | No cache | L1 = 16kB<br />L2 = 128kB  |
|----------|:----:|:-----:|:--------:|:--------------------------:|
| Hit Time |   2  |  10   |    100   |  2 for L1<br />12 for L2   |
| Hit Rate |  90% | 97.5% |   100%   | 90% for L1<br />75% for L2 |
| AMAT     |  12  |  12.5 |    100   |         5.5                |
| (calc))  | \\(2+0.1\*100\\) | \\(10+0.025\*100\\) | \\(100+0\*100\\) | \\(2+0.1\*(10+0.25*100)\\)

Combining caches like this provide much better overall performance than just considering hit time and size.

### Hit Rate in L2, L3, etc.
When L2 cache is used alone, it has a higher hit rate (97.5% vs 75%) - so it looks like it has a lower hit rate, which is misleading.

75% is the "local hit rate", which is a hit rate that the cache actually observes. In this case, 90% of accesses were hits in L1, so L2 cache never observed those (which would have been a hit).

97.5% is the "global hit rate", which is the overall hit rate for any access to the cache.

### Global vs. Local Hit Rate
- Global Hit Rate:  1 - Global Miss Rate
- Global Miss Rate: \\(\frac{\text{# of misses in this cache}}{\text{# of all memory references}}\\)
- Local Hit Rate: \\(\frac{\text{# of hits}}{\text{# of accesses to this cache}}\\)
- Misses per 1000 instructions (MPKI)
  - Similar to Miss Rate, but instead of being based on memory references, it normalizes based on number of instructions

### Inclusion Property
- Block is in L1 Cache
  - May or may not ben in L2
  - Has to also be in L2 (Inclusion)
  - Cannot also be in L2 (Exclusion)

When L1 is a hit, LRU counters on L2 are not changed - over time, L2 may decide to replace blocks that are still frequently accessed in L1. So Inclusion is not guaranteed. One way to enforce it is an "inclusion bit" that can be set to keep it from being replaced.

[🎥 See Lecture Example (4:44)](https://www.youtube.com/watch?v=J8DQG9Pvp3U)

Helpful explanation in lecture notes about why Inclusion is desirable:

> Student Question: What is the point of inclusion in a multi-level cache? Why would the effort/cost be spent to try and enforce an inclusion property?
> 
> I would have guessed that EXCLUSION would be a better thing to work towards, get more data into some part of the cache. I just don't get why you would want duplicate data taking up valuable cache space, no matter what level.
> 
> Instructor Answer: Inclusion makes several things simpler. When doing a write-back from L1, for example, inclusion ensures that the write-back is a L2 hit. Why is this useful? Well, it limits how much buffering we need and how complicated things will be. If the L1 cache is write-through, inclusion ensures that a write that is a L1 hit will actually happen in L2 (not be an L2 miss). And for coherence with private L1 and L2 caches (a la Intel's i3/i5/i7), inclusion allows the L2 cache to filter requests from other processors. With inclusion, if the request from another processor does not match anything in our L2, we know we don't have that block. Without inclusion, even if the block does not match in L2, we still need to probe in the L1 because it might be there.

*[AMAT]: Average Memory Access Time
*[LRU]: Least-Recently Used
*[MPKI]: Misses Per 1000 Instructions
*[MSHR]: Miss Status Handling Register
*[MSHRs]: Miss Status Handling Registers
*[NMRU]: Not-Most-Recently Used
*[PIPT]: Physically Indexed Physically Tagged
*[PLRU]: Pseudo-Least Recently Used
*[RWX]: Read, Write, Execute
*[SA]: Set-Associative
*[TLB]: Translation Look-Aside Buffer
*[VIPT]: Virtually Indexed Physically Tagged
*[VIVT]: Virtual Indexed Virtually Tagged