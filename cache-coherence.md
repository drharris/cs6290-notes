---
id: cache-coherence
title: Cache Coherence
sidebar_label: Cache Coherence
---

[🔗Lecture on Udacity (2hr)](https://classroom.udacity.com/courses/ud007/lessons/907008654/concepts/last-viewed)

## Cache Coherence Problem
![cache coherence problem](https://i.imgur.com/VyEGRMt.png)
As each core reads from memory, it may be using the cache, and value changes on other cores do not properly update all cores. Thus, it may operate on incorrect values with many reads and writes. This is called an Incoherent state. We instead need the idea of Cache Coherence - even when the cache is spread among many cores, it should behave as a single memory.

## Coherence Definition
1) Read R from address X on core C1 returns the value written by the most recent write W to X on C1, if no other core has written to X between W and R.
   - Case when only one core is operating on a location, should behave like a uniprocessor.
2) If C1 writes to X and C2 reads after a sufficient time, and there are no other writes in-between, C2's read returns the the value from C1's write.
   - After a "sufficent time", a core must read the "new" values produced by other cores using that location.
   - This requires some kind of active design to make this work correctly.
3) Writes to the same location are serialized: any two writes to X must be seen to occur in the same order on all cores.
   - All cores should agree on which values were written in a particular order.

## How to Get Coherence
Options:
1) No caches (really bad performance)
2) All cores share same L1 cache (bad performance)
3) ~~Private write-through caches~~
   (not really coherent because it doesn't prevent stale reads)
4) Force read in one cache to see write made in another.
   1) Pick one of these:
      * Broadcast writes to update other caches (Write-Update Coherence)
      * Writes prevent hits to other copies (Write-Invalidate Coherence)
   2) And, pick one of these:
      * All writes are broadcast on shared bus (Snooping) - bus can be a bottleneck
      * Each block assigned an "Ordering Point" (Directory)

## Write-Update Snooping Coherence
Two caches with valid bit (V), tag (T) and data, both connected to the same bus with memory. We have the following instructions:
```
    Core 0        Core 1
1   RD A          WR A=1
2   WR A=2        WR A=3
```
The first instruction executes with the Cache 0, is a miss, and is pulled from memory (A=0). Cache 1 could see this but does not care since it only updates on writes. Cache 1 gets the other instruction, writes to A in its own cache, and puts the request on the bus. It is seen by Cache 0, which then updates itself with the new value.

Then, we get the second instructions both executing (close to) simultaneously. With a single bus, only one of these can send on the bus at the same time and must arbitrate to see which "wins". So both modify their own cache at the same time (Cache 0: A=2, Cache 1: A=3). Let's say Core 0 wins the arbitration and sends A=2 on the bus. Cache 1 then updates to A=2, and _then_ sends its own A=3 on the bus, updating both caches.

In this way, at any point in time all processors agree on the historical values of A in the same orders, because the bus is serialized.

![write update snooping coherence](https://i.imgur.com/bhcMm0n.png)

### Write-Update Optimization 1: Memory Writes
Memory is on the same bus, yet is very slow, and thus it becomes a big bottleneck as it cannot keep up with all the writes. We want to thus avoid unneccessary memory writes. The solution is as we have seen before - add a "dirty" bit to each cache. The cache with a dirty bit set will then be responsible for answering any requests to memory. If a cache with dirty bit set snoops a write on the bus, it unsets the dirty bit and is no longer responsible. Memory is only written to when a block in cache with its dirty bit set is replaced. Thus, the last writer is responsible for writing to memory. 

* Dirty Bit benefits:
  * Write to memory only when block replaced
  * Read from memory only if no cache has that block in a dirty state.

### Write-Update Optimization 2: Bus Writes
With the previous optimization, memory is no longer as busy, but the bus still gets all writes, and now becomes the bottleneck in the system. This write does have to happen, because this is a Write-Update coherence system. But, what about the writes that aren't in any other cache? Those updates are wasted.

We can add a "Shared" bit to the block in cache that tells us if others are using this block or not. Additionally, there is another line on the bus pulled high on a memory read, so that each core can easily tell if other caches are using a block. If so, we can update the Shared bit to 1 on read. A cache can also pull it high if it sees a write on a block it already has in cache, to signal to the other cache that it is being used so it can mark it's Shared bit.

When a Write happens on a cache with Shared=1, it knows to broadcast that write on a bus, otherwise it will not. In this way, we get the Write-Update behavior when there is sharing, but otherwise we do not add extra traffic to the shared bus.

## Write-Invalidate Snooping Coherence
This works similar to Write-Update with both opimizations (dirty/shared) applied. However, it differs in that instead of broadcasting the entire new value on every write, we only broadcast the fact that tag has been modified. Other caches containing that tag can simply unset their Valid bit. The next time that block is read from cache, it will force that cache to re-read, which is then answered by the cache with Dirty=0 or else the Memory.

The disadvantage with Write-Invalidate is that all readers have a cache miss when reading something previously written on another cache. But the advantage is that multiple writes don't force caches to continue updating blocks they may not need. Additionally, we know when a write is sending an invalidation, we can then unset the Shared bit (since we know all caches will have to re-read that block at some point), and thus reduce bus activity further.

## Update vs. Invalidate Coherence
| Application Does... | Update | Invalidate |
|---|---|---|
| Burst of writes to one address | Each write sends an update (bad) | First write invalidates, other accesses are just hits (good) |
| Write different words in same block | Update sent for each word (bad) | First write invalidates, other accesses are just hits (good) |
| Producer-Consumer WR then RD | Producer sends updates, consumer hits (good) | Producer invalidates, consumer misses and re-reads (bad) |

And, the winner is... Invalidate! Overall, Invalidate is just slightly better in regard to these activities, but where it really shines is:

| | | |
|---|---|---|
| Thread moves to another core | Keep updating the old core's cache (horrible) | First write to each block invalidates, then no traffic (good) |

## MSI Coherence
A block can be in one of three states: 
1. I(nvalid): (V=0, or not in cache)
   * Local Read: Move to Shared state, **put RD on bus**
   * Local Write: Move to Modified state, **put WR on bus**
   * Snoop RD/WR on bus: (Remain in Invalid state)
2. S(hared): (V=1, D=0)
   * Local Read: (Remain in Shared state)
   * Local Write: Move to Modified state, **Put Invalidation on Bus**
   * Snoop WR on bus: Move to Invalid state
   * Snoop RD on bus: (Remain in Shared state)
3. M(odified):  (V=1, D=1)
   * Local Read: (Remain in Modified state)
   * Local Write: (Remain in Modified state)
   * Snoop WR on bus: Move to Invalid state, **WR back**
     * (can delay WR and then proceed again once this WR is done)
   * Snoop RD on bus: Move to Shared state, **WR back**

![MSI coherence](https://i.imgur.com/L3zVF7o.png)

### Cache-to-Cache Transfers
* Core1 has block B in M state
* Core2 puts RD on bus
* Core1 has to provide data... but how?
  * Solution 1: Abort/Retry
    * Core1 cancels Core2's request ("abort" signal on bus)
    * Core1 does normal write-back to memory
    * Core2 retries, gets data from memory
    * Problem: double the memory latency (write and read)
  * Solution 2: Intervention
    * Core1 tells memory it will supply the data ("intervention" signal on bus)
    * Core1 responds with data
    * Memory picks up data (both caches are now in shared state and think the block is not dirty, so memory also needs to update)
    * Problem: more complex hardware
  * Most use some form of Intervention, to avoid performance hit.

### Avoiding Memory Writes on Cache-to-Cache Transfers
* C1 has block in M state
* C2 wants to read, C1 responds with data -> C1: S, C2: S
* C2 writes -> C1: I, C2: M
* Maybe this repeats a few times, but memory write happens every time C1 responds with data
* C1 read, C2 responds with data -> C1: S, C2: S
* C3 read, memory provides data (memory read, even though either cache could respond)
* C4 read, memory provides data (...)

So, we want to avoid these memory read/writes if another cache already has the data. 

We need to make a non-M version responsible for:
* Giving data to other caches
* Eventually writing block to memory

We need to know which cache is "responsible" for the data. New State: O(wned).

## MOSI Coherence
* Like MSI, except:
  * M => snoop A Read => O (not S)
    * Memory does not get accessed!
  * O State, like S, except
    * Snoop a read => Provide Data
    * Write-Back to memory if replaced
* M: Exclusive Read/Write Access, Dirty
* S: Shared Read Access, Clean
* O: Shared Read Access, Dirty (only one block in this state)

### M(O)SI Inefficiency
* Thread-Private Data
  * All Data in a single-threaded program
  * Stack in Multi-threaded programs
* Data Read, then Write
  * I -> Miss -> S -> Invalidation -> M
  * Uniprocessor: V=0 => Read - Miss => V=1 => Hit => D=1
  * We want to avoid this Invalidation step that is unnecessary for thread-private data. For this, we will add another state: (E)xclusive

## The E State
* M: Exclusive Access (RD/WR), Dirty
* S: Shared Access (RD), Clean
* O: Shared Access (RD), Dirty
* E: Exclusive Access (RD/WR), Clean

For the Read/Write loop, here are how each model works (with bus traffic required)

|        |         MSI       |        MOSI       |        MESI       |       MOESI       |
|--------|:-----------------:|:-----------------:|:-----------------:|:-----------------:|
| `RD A` | I -> S<br/>(miss) | I -> S<br/>(miss) | I -> E<br/>(miss) | I -> E<br/>(miss) |
| `WR A` | S -> M<br/>(inv)  | S -> M<br/>(inv)  | E -> M            | E -> M            |

So, with MESI/MOESI, we get the same behavior as if we had used the same sequence of accesses on a uniprocessor. The E state allows us to achieve a cache hit we're looking for.

## Directory-Based Coherence
* Snooping: broadcast requests so others see them, and to establish ordering
  * Bus becomes a bottleneck
  * Snooping does not work well with > 8-16+ cores
* Non-Broadcast Network
  * How do we observe requests we need to see?
  * How do we order requests to the same block?

### Directory
* Distributed across cores
  * Each "slice" serves a set of blocks
    * One entry for each block it serves
    * Entry tracks which caches have block (in non-I state)
    * Order of accesses determined by "home" slice
  * Caches still have same states: MOESI
    * When we send a request to read or write, it no longer gets broadcast on a bus, it is communicated to a single directory.

### Directory Entry
* 1 Dirty Bit (causes us to find out if a cache needs to do a write-back)
* 1 Bit per Cache: present in that cache

In this example with 8 cores, a read request on Cache 0 of B would be sent to the home slice for block B (instead of being broadcast on a bus). This directory responds with the data (from memory), and that it has Exclusive access. The directory then sets bits for Dirty and Presence[0]. 

Cache 1 then performs a write request of B. The directory forwards that write to Cache 0, which moves to Invalid state and can then either return the data (since in E state), or just ignore and acknowledge the invalidation. The directory unsets bit Presence[0] (because it is in I state), sets bit Presence[1] and Cache 1 moves to the M state.
![directory entry](https://i.imgur.com/pfSVrbT.png)

### Directory Example
[🎥 View lecture video (4:59)](https://www.youtube.com/watch?v=lZZYILcQ68Y)

## Cache Misses with Coherence
* 3 Cs: Compulsory, Conflict, Capacity
* Another C: Coherence Miss
  * Example: If we read something, somebody else writes it, and we read it again
* So, 4 Cs now!
* Two types of coherence misses:
  * True Sharing: Different cores access same data 
  * False Sharing: Different cores access different data, _but in the same block_

