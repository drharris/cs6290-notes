---
id: synchronization
title: Synchronization
sidebar_label: Synchronization
---

[🔗Lecture on Udacity (1hr)](https://classroom.udacity.com/courses/ud007/lessons/906999159/concepts/last-viewed)

## Synchronization Example
In the following example, each thread is counting letters on its own half of the document. Most of the time, this can work out ok as each thread is operating on different parts of the document and things are being accessed in different orders depending on the letters each thread encounters. But what if both threads simultaneously attempt to update the counter for letter 'A'? Both `LW` for the same address for the proper counter, increments that value, and stores it. In the end, the counter only increments once instead of twice. We need a way to separate lines 2-4 into an Atomic Section (Critical Section) that cannot run simultaneously. With such a construct, one thread completes and stores the incremented value, and cache coherence ensures the other thread obtains the newly updated value before incrementing it.

![synchronization example](https://i.imgur.com/5viRJgS.png)

### Synchronization Example - Lock
The type of synchronization we use for atomic/critical sections is called Mutual Exclusion (Lock). When entering a critical section, we request a lock on some variable, and then unlock it once complete. The lock construct ensures that only one thread can lock it at a time, in no particular order - whichever thread is able to lock it first. The code above may now look like

```mipsasm
    LW     L, 0(R1)
    lock   CountLock[L]
    LW     R, Count[L]
    ADDI   R, R, 1
    SW     R, Count[L]
    unlock CountLock[L]
```

## Lock Synchronization
Naive implementation:
```cpp
 1   typedef int mutex_type;
 2   void lock_init(mutex_type &lockvar) {
 3       lockvar = 0;
 4   }
 5   void lock(mutex_type &lockvar) {
 6       while(lockvar == 1);
 7       lockvar = 1;
 8   }
 9   void unlock(mutex_type &lockvar) {
10       lockvar = 0;
11   }
```
This implementation has an issue in that if both threads attempt to access line 6 (`while`) simulatanously while `lockvar` is 0, both may wind up in the critical section at the same time. So really, lines 6 and 7 need to be in an atomic section of its own! So it seems there is a paradox, in that we need locks to implement locks.

## Implementing `lock()`
```cpp
 1   void lock(mutex_type &lockvar) {
 2   Wait:
 3      lock_magic();
 4      if(lockvar == 0) {
 5          lockvar = 1;
 6          goto Exit;
 7      }
 8      unlock_magic();
 9      goto Wait;
10   Exit:
11   }
```

But, we know there is no magic.
Options:
- Lamport's Bakery Algorithm
  - Complicated, expensive (makes lock slow)
- Special atomic read/write instructions
  - We need an instruction that both reads and writes memory

## Atomic Instructions
- Atomic Exchange
  - Swaps contents of the two registers
  - `EXCH R1, 78(R2)` allows us to implement lock:
    ```cpp
    R1 = 1;
    while(R1 == 1)
        EXCH R1, lockvar;
    ```
  - If at any point lockvar becomes 0, we swap it with R1 and exit the while loop. If other threads are attempting the lock at the same time, they will get our thread's R1.
  - Drawback: writes all the time, even while the lock is busy
- Test-and-Write
  - We test a location, and if it meets some sort of condition, we then do the write.
  - `TSTSW R1, 78(R2)` works like:
    ```cpp
    if(Mem[78+R2] == 0) {
        Mem[78+R2] = R1;
        R1 = 1;
    } else {
        R1 = 0;
    }
    ```
  - Implement lock by doing:
    ```cpp
    do {
        R1=1;
        TSTSW R1, lockvar;
    } while(R1==0);
    ```
  - This is good because we only do the write (and thus cause cache invalidations) when the condition is met. So, there is only communication happening when locks are acquired or freed.
- Load Linked/Store Conditional

## Load Linked/Store Conditional (LL/SC)
- Atomic Read/Write in same instruction
  - Bad for pipelining (Fetch, Decode, ALU, Mem, Write) - add a Mem2, Mem3 stage to handle all reads/writes needed
- Separate into 2 instructions
  - Load Linked
    - Like a normal `LW`
    - Save address in Link register
  - Store Conditional
    - Check if address is same as in Link register
      - Yes? Normal `SW`, return 1
      - No? Return 0!

### How is LL/SC Atomic?
The key is that if some other thread manages to write to lockvar in-between the `LL`/`SC`, the link register will be 0 and the `SC` will fail, due to cache coherence. A major benefit here is that simple critical sections no longer need locks, as we can `LL`/`SC` directly on the variable.

![how is ll/sc atomic](https://i.imgur.com/Q5NjAIk.png)

## Locks and Performance
[🎥 View lecture video (3:20)](https://www.youtube.com/watch?v=JS88digI8iQ)

Atomic Exchange has poor performance given that each core is constantly exchanging blocks, triggering invalidations on other cores. This level of overhead is very power hungry and slows down useful work that could be done.

### Test-and-Atomic-Op Lock
Recall original implementation of atomic exchange:

```cpp
R1 = 1;
while(R1 = 1)
    EXCH R1, lockvar;
```

We can improve this via normal loads and only using `EXCH` if the local read shows the lock is free. Now, the purpose of the exchange is only as a final way to safely obtain the lock.

```cpp
R1 = 1;
while(R1 == 1) {
    while(lockvar == 1);
    EXCH R1, lockvar;
}
```

This implementation allows `lockvar` to be cached locally until it is released by the core inside the critical section. This eliminates nearly all coherence traffic and leaves the bus free for the locked core to handle any coherence requests for `lockvar`.

## Barrier Synchronization
A barrier is a form of synchronization that waits for all threads to complete some work before proceeding further. An example of this is if the program is attempting to operate on data computed by multiple threads - all threads need to have completed their work before the thread(s) responsible for processing that work can continue.

- All threads must arrive at the barrier before any can leave.
- Two variables:
  - Counter (count arrived threads)
  - Flag (set when counter == N)

### Simple Barrier Implementation
```cpp
 1  lock(counterlock);
 2      if(count==0) release=0; // re-initalize release
 3      count++;                // count arrivals
 4  unlock(counterlock);
 5  if(count==total) {
 6      count=0;                // re-initialize barrier
 7      release=1;              // set flag to release all threads
 8  } else {
 9      spin(release==1);       // wait until release==1
10  }
```

This implementation has one major flaw. Upon the first barrier encounter, this works correctly. However, what if the work then continues for awhile and we try to go back to using these barrier variables for synchronization? We expect that we will start with `count == 0` and `release == 0`, but this may not be true...

### Simple Barrier Implementation Doesn't Work
The main issue with using this barrier implementation multiple times is that some threads may not pick up the release right away. Consider that the final thread to hit the barrier sets `release=1`, but maybe Thread 0 is off doing something else like handling an interrupt and doesn't see it. But the other threads have already been released and are not doing much work, and one could come back and hit the barrier again, setting `release=0`. Thread 0 is finally done with its work, and sees that `release==0` and continues to wait at the first barrier! Furthermore, the other threads are now waiting on `release==1` on the second instance of the barrier, and since Thread 0 is now locked up this will never happen, resulting in deadlock.

### Reusable Barrier
```cpp
1   localSense = !localSense;
2   lock(counterlock);
3       count++;
4       if(count==total) {
5           count=0;
6           release=localSense;
7       }
8   unlock(counterlock);
9   spin(release==localSense);
```

This implementation works because it functions as a sort of flip-flop on each barrier instance. We are never re-initializing `release`, but only `localSense`. So each time we hit the barrier it simply waits on the release to "flip", at which point it continues. So even if some threads continue work up until the next barrier before another thread is released from the barrier, then they must still wait on that thread to also arrive at that barrier.