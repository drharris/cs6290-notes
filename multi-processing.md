---
id: multi-processing
title: Multi-Processing
sidebar_label: Multi-Processing
---

[🔗Lecture on Udacity (1.25hr)](https://classroom.udacity.com/courses/ud007/lessons/1097109180/concepts/last-viewed)

## Flynn's Taxonomy of Parallel Machines
* How many instruction streams
* How many data streams

| | Instruction Streams | Data Streams | |
|:---:|:---:|:---:|---|
| SISD<br/>(uniprocessor) | 1 | 1 | Single Instruction Single Data |
| SIMD<br/>(vector, SSE/MMX) | 1 | > 1 | Single Instruction Multiple Data |
| MISD<br/>(stream processor?) | > 1 | 1 | Multiple Instruction Single Data<br/> (rare) |
| MIMD<br/>(multiprocessor) | > 1 | > 1 | Multiple Instruction Multiple Data<br/> (most today, multi-core)|

## Why Multiprocessors?
* Uniprocessors already \\(\approx\\) 4-wide
  * Diminishing returns from making it even wider
  * Faster?? \\(frequency \uparrow\ \Rightarrow voltage \uparrow\ \\Rightarrow power \uparrow^3\ \Rightarrow (fire)\\)
* But Moore's Law continues!
  * 2x transistors every 18 months
  * \\(\Rightarrow\ \approx\\)2x performance every 18 months (assuming we can use all the cores)

## Multiprocessor Needs Parallel Programs!
* Sequential (single-threaded) code is _a lot_ easier to develop
* Debugging parallel code is _much_ more difficult
* Performance scaling _very_ hard to get
  * (as we scale # cores, performance scales equally, linearly)
  * In reality, performance will improve to a point, but then levels off. We can keep improving the program, but it will still max out at some level of performance no matter how many cores.

## Centralized Shared Memory
Each core has its own cache, but connected to the same bus that shares main memory and I/O. Cores can send data to each other by writing to a location in main memory. Similarly they can share I/O. This architecture is called a Symmetric Multiprocessor (SMP), another concept is Uniform Memory Access (UMA), in terms of time.
![centralized shared memory](https://i.imgur.com/27aePNH.png)

### Centralized Main Memory Problems
* Memory Size
  * Need large memory \\(\Rightarrow\\) slow memory
* Memory Bandwidth
  * Misses from all cores \\(\Rightarrow\\) memory bandwidth contention
    * As we add more cores, it starts to serialize against memory accesses, so we lose performance benefit

Works well only for smaller machines (2, 4, 8, 16 cores)

## Distributed ~~Shared~~ Memory
This uses message passing for a core to access another core's memory. If accesses are not in the local cache or the memory slice local to the core, it actually sends a request for this memory through the network to the core that has it. This system is also called a cluster or a multi-computer, since each core is somewhat like an individual computer.
* ![distributed memory](https://i.imgur.com/jIRNFwL.png)

These computers tend to scale better - not because network communication is faster, but rather because the programmer is forced to explicitly think about distributed memory access and will tend to minimize communication.

## A Message-Passing Program
```cpp
#define ASIZE 1024
#define NUMPROC 4
double myArray[ASIZE/NUMPROC]; // each processor allocates only 1/4 of entire array (assumes elements have been distributed somehow)
double mySum = 0;
for(int i=0; i < ASIZE/NUMPROC; i++)
    mySum += myArray[i];
if(myPID==0) {
    for(int p=1; p < NUMPROC; p++) {
        int pSum;
        recv(p, pSum);  // receives the sums from the other processors
        mySum += pSum;
    }
    printf("Sum: %lf\n", mySum);
} else {
    send(0, mySum);     // sends the local processor's sum to p0
}
```

## A Shared Memory Program
Benefit: No need to distribute the array!

```cpp
#define ASIZE 1024
#define NUMPROC 4
shared double array[ASIZE]; // each processor needs the whole array, but it is shared
shared double allSum = 0;   // total sum in shared memory
shared mutex sumLock;       // shared mutex for locking
double mySum = 0;
for(int i=myPID*ASIZE/NUMPROC; i < (myPID+1)*ASIZE/NUMPROC; i++) {
    mySum += array[i];      // sums up this processor's own part of the array
}
lock(sumLock);              // lock mutex for shared memory access
allSum += mySum;
unlock(sumLock);            // unlock mutex
// <<<<< Need a barrier here to prevent p0 from displaying result until other processes finish
if(myPID==0) {
    printf("Sum: %lf\n", allSum);
}
```

## Message Passing vs Shared Memory

|                   | Message Passing | Shared Memory |
|-------------------|:---------------:|:-------------:|
| Communication     |    Programmer   |   Automatic   |
| Data Distribution |      Manual     |   Automatic   |
| Hardware Support  |      Simple     |   Extensive   |
| Programming       |
| - Correctness     |    Difficult    | Less Difficult |
| - Performance     |    Difficult    | Very Difficult |

Key difference: Shared memory is easier to get a correct solution, but there may be a lot of issues making it perform well. With Message Passing, typically once you've gotten a correct solution you also have solved a large part of the performance problem.

## Shared Memory Hardware
* Multiple cores share physical address space
  * (all can issue addresses to any memory address)
  * Examples: UMA, NUMA
* Multi-threading by time-sharing a core
  * Same core \\(\rightarrow\\) same physical memory
  * (not really benefiting from multi-threading)
* Hardware multithreading in a core
  * Coarse-Grain: change thread every few cycles
  * Fine-Grain: change thread every cycle (more HW support)
  * Simultaneous Multi-Threading (SMT): any given cycle we can be doing multiple instructions from different threads
    * Also called Hyperthreading

## Multi-Threading Performance
[🎥 View lecture video (8:25)](https://www.youtube.com/watch?v=ZpqeeHFWxes)

Without multi-threading, processor activity in terms of width is very limited - there will be many waits in which no work can be done. As it switches from thread to thread (time sharing), the activity of each thread is somewhat sparse.

In a Chip Multiprocessor (CMP), each core runs different threads. While the activity happens simultaneously in time, each core still has to deal with the sparse workload created by the threads it is running. (Total cost 2x)

With Fine-Grain Multithreading, we have one core, but with separate sets of registers for different threads. Each thread's work switches from cycle to cycle, taking advantage of the periods of time in which a thread is waiting on something to be ready to do the next work. This keeps the core busier overall, at the slight expense of extra hardware requirements. (Total cost \\(\approx\\) 1.05x)

With SMT, we can go one step further by mixing instructions from different threads into the same cycle. This dramatically reduces the total idle time of the processor. This requires much more hardware to support. (Total cost \\(\approx\\) 1.1x)

## SMT Hardware Changes
* Fetch
  * Need additional PC to keep track of another thread
* Decode
  * No changes needed
* Rename
  * Renamer does the same thing as per one thread
  * Need another RAT for the other thread
* Dispatch/Execution
  * No changes needed
* ROB
  * Need a separate ROB with its own commit point
  * Alternatively: interleave in single ROB to save HW expense (more often in practice)
* Commit
  * Need a separate ARF for the other thread.
  ![smt hardware changes](https://i.imgur.com/4fC0ULd.png)

Overall: cost is not that much higher - the most expensive parts do not need duplication to support SMT.

## SMT, D$, TLB
With a VIVT cache, we simply send the virtual address into the cache and use the data that comes out. However, each thread may have different address spaces, so the cache has no clue which of the two it is getting. So with VIVT, we risk getting the wrong data!

With a VIPT cache (w/o aliasing), the same virtual address will be combined with the physical address and things will work correctly as long as the TLB provides the right information. So for VIPT and PIPT both, SMT addressing will work fine as long as the TLB is thread-aware. One way to ensure this is with a thread bit on each TLB entry, and to pass the thread ID into the TLB lookup.
![SMT, cache, TLB](https://i.imgur.com/6zKXp5i.png)

## SMT and Cache Performance
* Cache on the core is shared by all SMT threads
* Fast data sharing (good)
  ```
  Thread 0: SW A
  Thread 1: LW A <--- really good performance
  ```
* Cache capacity (and associativity) shared (bad)
  * If WS(TH0) + WS(TH1) - WS(TH0, TH1) > D$ size:    
    \\(\Rightarrow\\) Cache Thrashing
  * If WS(TH0) < D$ size    
    \\(\Rightarrow\\) SMT performance can be worse than one-at-a-time
  * If the threads share a lot of data and fit into cache, maybe SMT performance doesn't have as many issues.



*[CMP]: Chip Multiprocessor
*[ARF]: Architectural Register File
*[SMT]: Simultaneous Multi-Threading
*[SMP]: Symmetric Multiprocessor
*[WS]: Working Set
*[D$]: Data Cache
*[PIPT]: Physically Indexed Physically Tagged
*[TLB]: Translation Look-Aside Buffer
*[VIPT]: Virtually Indexed Physically Tagged
*[VIVT]: Virtual Indexed Virtually Tagged
*[ALU]: Arithmetic Logic Unit
*[IQ]: Instruction Queue
*[LB]: Load Buffer
*[LW]: Load Word
*[PC]: Program Counter
*[RAT]: Register Allocation Table (Register Alias Table)
*[ROB]: ReOrder Buffer
*[SB]: Store Buffer
*[SW]: Store Word
*[UMA]: Uniform Memory Access
*[NUMA]: Non-Uniform Memory Access