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

## RAT Updates on Commit

[🎥 Video explanation](https://www.youtube.com/watch?v=PYFg7QOfcvI). This example walks through how the ROB, RAT, and REGS work together during the commit phase. At a basic level, as instructions in the ROB are completed, it updates the register values accordingly. However, it will only update the RAT (to change its pointer from a ROB entry to a Register) if it just committed the ROB instruction the RAT entry was pointing to. For example, if the RAT entry for `R3` was pointing to `ROB2`, when `ROB2` executes it will commit the update to the associated register, and also update the RAT entry `R3` to point to `R3` now. In this way, registers and RAT are both kept updated only when the instruction is finally committed.

## ROB Example
_These examples are best viewed as videos, so links are below..._

1. [🎥 Cycles 1-2](https://www.youtube.com/watch?v=39AFF5Qq5DI)
2. [🎥 Cycles 3-4](https://www.youtube.com/watch?v=c3hFm_DOUA0)
3. [🎥 Cycles 5-6](https://www.youtube.com/watch?v=4nZN_mLcCJo) 
4. _(cycles 7-12 are "fast forwarded")_
5. [🎥 Cycles 13-24](https://www.youtube.com/watch?v=bE3IFvoChyw)
6. [🎥 Cycles 25-43](https://www.youtube.com/watch?v=HmURweRTsU4)
7. [🎥 Cycles 44-48](https://www.youtube.com/watch?v=V0nywwV0lKU)
8. [🎥 Timing Example](https://www.youtube.com/watch?v=f9IcEtKTz8k)

## Unified Reservation Stations

With separate RS for separate units (e.g. ADD, MUL), often running out of RS spots on one unit will prevent the other unit from being issued too (because instructions must be issued in order). The structures themselves are functionally the same, so all RS can be unified into one larger array, to allow more total instructions to be issued. However, this requires additional logic in the dispatch unit to target the correct unit. But, reservation stations are expensive, so it may be better to add the additional logic rather than having stations go unused.

## Superscalar

Previous examples were limited to one instruction per cycle. For superscalar, we need to consider the following:
* Fetch > 1 inst/cycle
* Decode > 1 inst/cycle
* Issue > 1 inst/cycle (still should be in order)
* Dispatch > 1 inst/cycle
  * May require multiple units of each functional type
* Broadcast > 1 result/cycle
  * This involves not only having more buses for each result, but every RS has to compare with every bus each cycle
* Commit > 1 inst/cycle
  * Must still obey rule of in-order commits

With all of these, we must consider the "weakest link". If all of these are very large but one is limited to 3 inst/cycle, that will be the bottleneck in the pipeline.

## Termninology Confusion

| Academics | Companies, other papers |
| --- | --- |
| Issue | Issue, Allocate, Dispatch |
| Dispatch | Execute, Issue, Dispatch |
| Commit | Commit, Complete, Retire, Graduate |

So, it's complicated.

## Out of Order

In an out-of-order processor, not ALL pipeline stages are processing instructions out of order. Some stages must still be in-order to preserve proper dependencies.

| Stage | Order |
| --- | --- |
| Fetch | In-Order |
| Decode | In-Order |
| Issue | In-Order |
| Execute | Out-of-Order |
| Write/Bcast | Out-of-Order |
| Commit | In-Order |


## Additional Resources

From TA Nolan, here is an attempt to document how a CPU with ROB works:

```
While there is an instruction to issue
	If there is an empty ROB entry and an empty appropriate RS
		Put opcode into RS.
		Put ROB entry number into RS.
		For each operand which is a register
			If there is a ROB entry number in the RAT for that register
				Put the ROB entry number into the RS as an operand.
			else
				Put the register value into the RS as an operand.
		Put opcode into ROB entry.
		Put destination register name into ROB entry.
		Put ROB entry number into RAT entry for the destination register.
		Take the instruction out of the instruction window.
		
For each RS
	If RS has instruction with actual values for operands
		If an appropriate ALU or processing unit is free
			Dispatch the instruction, including operands and the ROB entry number.
			Free the RS.

While the next ROB entry to be retired has a result
	Write the result to the register.
	If the ROB entry number is in the RAT, remove it.
	Free the ROB entry.

For each ALU
	If the instruction is complete
		Put the result into the ROB entry corresponding to the destination register.
		For each RS waiting for it
			Put the result into the RS as an operand.
		Free the ALU.
```




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