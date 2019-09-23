---
id: reorder-buffer
title: ReOrder Buffer
sidebar_label: ReOrder Buffer
---

[🔗Lecture on Udacity](https://classroom.udacity.com/courses/ud007/lessons/945398787/concepts/last-viewed)

## Exceptions in Out Of Order Exceptions

Consider this set of instructions as a result of Tomasulo:

| &nbsp; | Instruction | Issue | Disp | WR |
|---|---|---|---|---|
| 1 | `DIV F10, F0, F6` | 1 | 2 | 42 |
| 2 | `L.D F2, 45(R3)`  | 2 | 3 | 13 | 
| 3 | `MUL F0, F2, F4`  | 3 | 14 | 19 |
| 4 | `SUB F8, F2, F6`  | 4 | 14 | 15 |

Let's say in instruction 1, the F6 == 0 and causes a Divide By Zero exception in cycle 40. Normally an exception would cause PC to be saved and then it would go to an exception handler, and then come back and operations would resume. However, in this case, by the time the exception occurs, F0 has been overwritten by instruction 3 and instruction 1 would produce an incorrect result. This can also happen if a Load has a page fault, and various other causes. 

## Branch Misprediction in OOO Execution

Similarly to before, what happens if branch misprediction has caused a register to be overwritten before another instruction completes?

```mipsasm
    DIV R1, R3, R4
    BEQ R1, R2, Label
    ADD R3, R4, R5
    ...
    DIV ...
Label:
    ...
```

If `DIV` takes many cycles, it will take a long time before we realize the branch was mispredicted. In the meantime, the third instruction may have completed and updated `R3`. When we realize the misprediction, we should behave as if the wrong branch had never executed, but this is now impossible.

Another issue is with Phantom Exceptions. Let's say the branch was mispredicted, and the program went on and the final DIV statement caused an exception - it could be in the exception handler before we even know the branch was mispredicted and the statement never should have run.

## Correct OOO Execution
- Execute OOO
- Broadcast OOO
- But deposit values to registers In-Order!

We need a structure called a ReOrder Buffer:
- Remembers program order
- Keeps results until safe to write to registers

## ReOrder Buffer (ROB)

This structure contains the register that will be updated, the value it should be updated to, and whether the corresponding instruction is really done and ready to be written. The instructions are in program order.

![ReOrder Buffer](https://i.imgur.com/vFBCy96.png)

There are also pointers to the next issuable instruction, and the next value to be committed (creating a standard FIFO buffer structure via head/tail pointers).

[🎥 How ROB fits into Tomasulo](https://www.youtube.com/watch?v=0w6lXz71eJ8) | 
[🎥 Committing a WR to registers](https://www.youtube.com/watch?v=p6clkAsUV7E)

Basically, it takes the place of the link between the RAT and RS, and now both point to this intermediary ROB structure. So RS now only is concerned with dispatching instructions, and doesn't have to wait for the result to be broadcast before freeing a spot in the RS (because the RS itself is not a register alias to be resolved)

## Hardware Organization with ROB

Things are much the same as before. We still have an IQ that feeds into one or more RS, and we have a RAT that may have entries pointing to an RF. The primary difference is where before the RAT may have pointed to the RS, it will now point to the ROB instead. On each cycle, the ROB may commit its result to the RF and RS if the associated instruction is considered "done".

## Branch Misprediction Recovery

Describing how ROB works on a branch misprediction:
- [🎥 Part 1](https://www.youtube.com/watch?v=GG09xSZ32MU) | 
- [🎥 Part 2](https://www.youtube.com/watch?v=ozc8ceuw-GA)

The summary is that once the commit pointer reaches a mispredicted branch, we simply empty the ROB (issue pointer = commit pointer). The register file contains the correct registers. The RAT is modified to point to the associated registers. And finally, the next fetch will fetch from the correct PC. Therefore, the ROB makes recovery from a mispredicted branch fairly straightforward.

## ROB and Exceptions

Reminder: two scenarios:
1. Exception in long-running instruction (or page fault)
   - Simply flush the ROB and move to exception handler
   - ROB ensures that no wrong registers have been committed and the program is in a good state for the exception handler
2. Phantom exceptions for mispredicted branch
   - The instruction with an exception is tagged as an exception in the ROB
   - When we figure out the branch is mispredicted it is handled like any other misprediction and thus the exception does not get handled (as it doesn't reach the commit point for the instruction with the exception)

## Commit == Outside view of "Executed"

[🎥 Video explanation](https://www.youtube.com/watch?v=vpPjDW48v90). The main point is that ROB makes it so the processor itself may see instructions executed in any order with mispredicted branches, but the programmer's view of the program's execution is that all instructions are committed in order with no wrong branches taken.


*[ALU]: Arithmetic Logic Unit
*[CPI]: Cycles Per Instruction
*[ILP]: Instruction-Level Parallelism
*[IPC]: Instructions per Cycle
*[IQ]: Instruction Queue
*[LB]: Load Buffer
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