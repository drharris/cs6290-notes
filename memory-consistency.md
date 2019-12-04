---
id: memory-consistency
title: Memory Consistency
sidebar_label: Memory Consistency
---

[🔗Lecture on Udacity (32 min)](https://classroom.udacity.com/courses/ud007/lessons/914198580/concepts/last-viewed)

## Memory Consistency
- Coherence: Defines order of accesses to **the same address**
  - We need this to share data among threads
  - Does not say anything about accesses to different locations
- Consistency: Defines order of accesses to **different addresses**
  - Does this even matter?

### Consistency Matters
Because of coherence, stores must always occur in-order. But, since out of order processors could change the order of loads, you might get results out of order that wouldn't be possible from the program logic alone.

Examples: [🎥 View lecture video (3:59)](https://www.youtube.com/watch?v=uh8gF64345I), [🎥 View quiz from lecture (3:33)](https://www.youtube.com/watch?v=4X-DciIfFcc)

### Why We Need Consistency
- Data-Ready Flag Synchronization
  - See quiz from previous section. Effects of branch prediction can cause loads to happen before the data is ready (according to some flag)
- Thread Termination
  - Thread A creates Thread B
  - Thread B works, updates data
  - Thread A waits until B exits (system call)
    - Branch prediction may cause this to execute before the following:
  - Thread B done, OS marks B done

We need an additional set of ordering restrictions, beyond coherence alone!

## Sequential Consistency
The result of any execution should be as if accesses executed by each processor were executed in-order, and accesses among different processors were arbitrarily interleaved.

- Simplest Implementation: remove the "as if"
  - A core performs next access only when ALL previous accesses are complete
  - This works fine, but it is really, really bad for performance (MLP = 1)

### Better Implementation of Sequential Consistency
- Core can reorder loads
- Detect when SC may be violated \\(\Rightarrow\\) Fix
  - How? In the ROB we have all instructions in program order, regardless of execution order.
    - If instructions actually executed in order, we're ok
    - If instructions are out of order but no other stores were done to any of the reordered addresses in the meantime, we're also ok.
      - Example: `LW A` and `LW B` are program order, but they get executed out of order. There is nothing such as a `SW B` after the `LW B`, so by the time we hit `LW A` it is still consistent with what it would have been in-order.
    - If we see a store after a load that is out of order (e.g. execution order of `LW B` then `SW B` then `LW A`), we then know that load has to be replayed, because it would have been a different value in program order. We know it is replayable because it is still beyond the commit point in the ROB, so we are capable of replaying it.
    - In order to detect such a violation, for anything we load out of order we need to monitor coherence traffic until we get back in order.

## Relaxed Consistency
An alternative approach to SC is to not set the expectation to the programmers for SC, but something slightly less strict.
- Four types of ordering
  1. `WR A` \\(\rightarrow\\) `WR B` 
  2. `WR A` \\(\rightarrow\\) `RD B`
  3. `RD A` \\(\rightarrow\\) `WR B`
  4. `RD A` \\(\rightarrow\\) `RD B`
- Sequential Consistency: All must be obeyed!
- Relaxed Consistency: some of these types need not be obeyed at all the times
  - Read/Read (#4) best example
  - How do we write correct programs if we cannot expect this ordering to be consistent?

### Relaxed Consistency implementation
- Allow reordering of "Normal" accesses
- Add special non-reorderable accesses (separate instructions than normal `LW`/`SW`)
  - Must use these when ordering in the program matters
- Example: x86 `MSYNC` instruction
  - Normally, all accesses may be reordered
  - But, no reordering across the `MSYNC`
    - [`LW`, `SW`, `LW`, `SW`] \\(\rightarrow\\) `MSYNC`, \\(\rightarrow\\) [`LW`, `SW`, ...]
    - Acts as a barrier between operations that expect things to be in order
    ```cpp
    while(!flag);
    MSYNC
    //use data -- ensures the data is correctly synchronized to the flag state regardless of reordering
    ```
  - When to `MSYNC` for synchronization? After a lock/"acquire", but before an unlock/"release"

## Data Races and Consistency
![data races and consistency](https://i.imgur.com/owEQKj4.png)

One key point is that when creating a data-race-free program, we may want to debug in a SC environment until proper synchronization ensures that we are free of any potential data races. Once we are confident of that, we can move to a more relaxed consistency model and rely on synchronization to ensure consistency is maintained. If the program is not data race free, then anything can happen!

## Consistency Models
- Sequential Consistency
- Relaxed Consistency
  - Weak (distinguishes between synchronization and non-synchronization accesses, and prevents any synchronization-related accesses from being reordered, but all others may be freely reordered)
  - Processor
  - Release (distinguishes between lock and unlock (acquire/release) accesses, and does not allow non-synchronization accesses to be reordered across this lock/unlock boundary)
  - Lazy Release
  - Scope
  - ...


*[MLP]: Memory Level Parallelism
*[SC]: Sequential Consistency
*[ROB]: Re-Order Buffer