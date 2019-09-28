---
id: memory-ordering
title: Memory Ordering
sidebar_label: Memory Ordering
---

[🔗Lecture on Udacity (30 min)](https://classroom.udacity.com/courses/ud007/lessons/937498641/concepts/last-viewed)

## Memory Access Ordering

With ROB and Tomasulo, we can enforce order of dependencies on registers between instructions. Is there an equivalent for memory acces (load/store)? Can they be reordered?

So far, we have:
- Eliminated Control Dependencies
  - Branch Prediction
- Eliminated False Dependencies on registers
  - Register Renaming
- Obey RAW register dependencies
  - Tomasulo-like scheduling

What about memory dependencies?
```mipsasm
    SW  R1, 0(R3)
    LW  R2, 0(R4)
```
In this case, if 0(R3) and 0(R4) point to different addresses, then order does not matter. But if they point to the same address, then we need a way to handle this dependency and keep them in order.

## When does Memory Write Happen?

At commit! Any instruction may execute but not be committed (due to exceptions, branch misprediction), so memory should not be written until the instruction is ready to be committed.

But if we write at the commit, where does a `LOAD` get its data?

## Load Store Queue (LSQ)

We need an LSQ to provide values from stores to loads, since the actual memory will not be written until a commit. We use the following structure:

| L/S | Addr | Val | C |
| --- | ---  | --- |---|
|  `` |      |     |   |

L/S: bit for whether it is load or store    
Addr: address being accessed
Val: value being stored or loaded
C: instruction has been completed

An example of using the LSQ:
1. [🎥 Part 1](https://www.youtube.com/watch?v=utRgthVxAYk)
2. [🎥 Part 2](https://www.youtube.com/watch?v=mbwf-CoA5Zg)

For each Load instruction, it looks in the LSQ to see if the value was already stored to that address by a previous instruction. If so, it uses "Store-to-Load Forwarding" to obtain the value for the Load. If not, it goes to memory. In some cases, a Store may not yet know its address (needs to be computed). In this case, a Load may have not known there was a previous store on that address and would have obtained an incorrect value. There are some options to handle this:
1. All mem instructions in-order (very low performance)
2. Wait for all previous store addresses to complete (still bad performance)
3. Go anyway - the problem will occur, but additional logic must be used to recover from this situation.

### Out-Of-Order Load Store Execution

| # | Inst | Notes |
|---| ---  | --- |
| 1 | `LOAD R3 = 0(R6)` | Dispatched immediately, but cache miss |
| 2 | `ADD R7 = R3 + R9` | Depends on #1, waiting... |
| 3 | `STORE R4 -> 0(R7)` | Depends on #2, waiting... |
| 4 | `SUB R1 = R1 - R2` | Dispatched immediately, R1 good |
| 5 | `LOAD R8 = 0(R1)` | R1 ready, so runs and cache hit |

In this example, instructions 1-3 are delayed due to a cache miss, but instructions 4 and 5 are able to execute. `R8` now contains the value from address `0(R1)`. Later, instructions 1-3 will execute, and address `0(R7)` contains the value of `R4`. If the addresses in `R1` and `R7` do not match, then this is ok. However, if they are the same number (given that `R7` was calculated in instruction 2 and we did not know this yet), then we have a problem, because R8 contains the previous value from that address, not the value it should have had with the ordering.

### In-Order Load Store Execution

Similar to previous example, we may execute some instructions out of order (#4 may execute right away), but all loads and stores are in order. A store instruction is considered "done" (for purposes of ordering) when the address is known, at which point it can check to ensure any other instructions do not conflict with it. Of course this is not very high-performance, as the final `LOAD` instruction is delayed considerably - and if it is a cache miss, then the performance takes an even greater hit.

## Store-to-Load Forwarding

Load:
- Which earlier `STORE` do we get value from?

Store:
- Which later `LOAD`s do I give my value to?

The LSQ is responsible for deciding these things.

## LSQ Example

[🎥 Example Video (5:29)](https://www.youtube.com/watch?v=eHVLMgfy-Jc)

A key point from this is the connection to Commits as we learned in the ROB lesson. The stores are only written to data cache when the associated instruction is committed - thereby ensuring that exceptions or mispredicted branches do not affect the memory itself. At any point execution could stop and the state of everything will be ok.

## LSQ, ROB, and RS

Load/Store:
- Issue
  - Need a ROB entry
  - Need an LSQ entry (similar to needing an RS in a non-Load/Store issue)
- Execute Load/Store:
  - Compute the Address
  - Produce the Value
  - _(Can happen simultaneously)_
- (LOAD only) Write-Result
  - And Broadcast it
- Commit Load/Store
  - Free ROB & LSQ entries
  - (STORE only) Send write to memory


*[ALU]: Arithmetic Logic Unit
*[CPI]: Cycles Per Instruction
*[ILP]: Instruction-Level Parallelism
*[IPC]: Instructions per Cycle
*[IQ]: Instruction Queue
*[LB]: Load Buffer
*[LSQ]: Load-Store Queue
*[LW]: Load Word
*[OOO]: Out Of Order
*[PC]: Program Counter
*[RAT]: Register Allocation Table (Register Alias Table)
*[RAW]: Read-After-Write
*[ROB]: ReOrder Buffer
*[SB]: Store Buffer
*[SW]: Store Word
*[WAR]: Write-After-Read
*[WAW]: Write-After-Write
*[RAR]: Read-After-Read