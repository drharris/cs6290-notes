---
id: instruction-scheduling
title: Instruction Scheduling
sidebar_label: Instruction Scheduling
---

[🔗Lecture on Udacity](https://classroom.udacity.com/courses/ud007/lessons/3643658790/concepts/last-viewed)

## Improving IPC
* ILP can be good, >> 1. 

* To get IPC \\(\approx\\) ILP, need to handle control dependencies (=> Branch Prediction).

* Need to eliminate WAR and WAW data dependencies (=> Register Renaming).

* Handle RAW data dependencies (=> Out of Order Execution)

* Structural Dependencies (=> Invest in Wider Issue)

## Tomasulo's Algorithm

Originall used in IBM 360, determines which instructions have inputs ready or not. Includes a form of Register Renaming, and is very similar to what is used today.

| Original Tomasulo | What is used today |
| --- | --- |
| Floating Point instructions | All instruction types |
| Fewer instructions in "window" | Hundreds of instructions considered |
| No exception handling | Explicit exception handling support |

### The Picture

![Tomasulo's Algorithm](https://i.imgur.com/MuCQEgr.png)

Instructions come from the Fetch unit in order and put into Instruction Queue (IQ). The next available instruction from the IQ is put into a Reservation Station (RS), where they sit until parameters become ready. There is a floating point register file (REGS), and these registers are put into the RS when they are ready. 

When an instruction is ready, it goes into the execution unit. Once the result is available it is broadcast onto a bus. This bus goes to the register file and also the RS (so they will be available when needed). Notice multiple inputs from this bus on the RS - this is because an instruction may need the value to be latched into both operands.

Finally, if the instruction deals with memory (load/store), it will go to the address generation unit (ADDR) for the address to be computed. It will then go to the Load Buffer (LB, provides just the address A) or Store Buffer (SB, provides both the data D and the address A). When it comes back from memory (MEM), its value is also broadcast on the bus, which also feeds into the SB (so stores can get their values when available).

Finally, the part where we send the instruction from the fetch to the memory or execution unit is called the "Issue". The place where the instruction is finally sent from the RS to execution is called the "Dispatch". And when the instruction is ready to broadcast its result, it's called a "Write Result" or "Broadcast".

## Issue

1. Take next (in program order) instruction from IQ
2. Determine where inputs come from (probably a RAT here)
   - register file
   - another instruction
3. Get free/available RS (of the correct kind)
   - If all RS are busy, we simply don't issue anything this cycle.
4. Put instruction in RS
5. Tag destination register of the instruction

### Issue Example

[🎥 Link to Video](https://www.youtube.com/watch?v=I2qMY0XvYHA) (best seen and not read)

![Issue Example](https://i.imgur.com/JdepPAx.png)

## Dispatch

[🎥 Link to Video](https://www.youtube.com/watch?v=bEB7sZTP8zc)

As RAT values from other instructions become available on the result broadcast bus, instructions are moved from the RS to the actual execution unit (e.g. ADD, MUL) to be executed.

### Dispatch - >1 ready

What happens if more than one instruction in the RS is ready to execute at the same time? How should Dispatch choose which to execute?
- Oldest first (easy to do, medium performance)
- Most dependencies first (hard to do, best performance)
- Random (easy to do, worst performance)

![Dispatch: >1 ready](https://i.imgur.com/uS0PRFt.png)

All are fine with respect to correctness because of RAT, but if we do not know the future we cannot predict most dependencies, even if that would yield the best overall performance due to possibly freeing up more other instructions to execute. Oldest first is the best compromise with performance and ease of implementation.

## Write Result (Broadcast)

Once the result is ready, it is put on the bus (with its RS tag). It is then used to update the register file and RAT, and then the RS is freed for the next instruction to use.

![Write Result](https://i.imgur.com/2lrRoFf.png)

### More than 1 Broadcast

What if in the previous example, the ADD and MUL complete simultaneously - which one is broadcast first?

Possibility 1: a separate bus for each unit. This allows both to be broadcast, but requires twice the comparators (every output from the bus now needs to consider both to select the correct tag).

Possibility 2: Select a higher priority unit based on some heuristic. For example, if one unit is slower, give it higher priority on the WR bus since it's likely the instruction has been waiting longer.

### Broadcast "Stale" Result

Consider a situation in which an instruction is ready to broadcast from RS4, but RS4 is nowhere in the RAT (perhaps overwritten by a new instruction that was both using and writing to the same register, now the RAT will reflect the new RS tag). What will happen during broadcast?

When updating the RS, it will work normally. RS4 will be replaced by the broadcasted value. In the RAT and RF, we do nothing. It is clear that it will never be used by future instructions, only ones currently in the RS.

## Tomasulo's Algorithm - Review

![Tomasulo's Algorithm - Review](https://i.imgur.com/tsOgTRq.png)

The key is that for each instruction it will follow the steps (issue, capture, dispatch, WR). Also each cycle, some instruction will be issued, another instruction will be captured, another dispatched, another broadcasting.

Because all of these things happen every cycle, we need to consider some things:
1. Can we do same-cycle issue->dispatch? No - on an issue we are writing to the RS, and that instruction is not yet recognizable as a dispatchable instruction.
2. Can we do same-cycle capture->dispatch? Typically No - on a capture the RS updates its status from "operands missing" to "operands available". But this is technically possible with more hardware.
3. Can we update RAT entry for both Issue and WR on same cycle? Yes - we just need to ensure that the one being issued is the end result of the RAT. The WR is only trying to point other instructions to the right register, which will also be updated this cycle.

## Load and Store Instructions

As we had dependencies with registers, we have dependencies through memory that must be obeyed or eliminated.
- RAW: SW to address, then LW from address
- WAR: LW then SW
- WAW: SW, SW to same address

What do we do in Tomasulo's Algorithm?
- Do loads and stores in-order
- Identify dependencies, reorder, etc. - more complicated than with registers, so Tomasulo chose not to do this.

## Tomasulo's Algorithm - Long Example
_These examples are best viewed as videos, so links are below..._

1. [🎥 Introduction](https://www.youtube.com/watch?v=2M5NQFAaILk)
2. [🎥 Cycles 1-2](https://www.youtube.com/watch?v=GC8Cp-M0o6Q)
3. [🎥 Cycles 3-4](https://www.youtube.com/watch?v=G0Kap6eq_Ys)
4. [🎥 Cycles 5-6](https://www.youtube.com/watch?v=I1VOoFhrnio)
5. [🎥 Cycles 7-9](https://www.youtube.com/watch?v=wmrPTJpUnV4)
6. [🎥 Cycles 10-end](https://www.youtube.com/watch?v=ZQ6Tdrs16_U)
7. [🎥 Timing Example](https://www.youtube.com/watch?v=ZqbhHjFSBoI)



*[ALU]: Arithmetic Logic Unit
*[CPI]: Cycles Per Instruction
*[ILP]: Instruction-Level Parallelism
*[IPC]: Instructions per Cycle
*[IQ]: Instruction Queue
*[LB]: Load Buffer
*[LW]: Load Word
*[PC]: Program Counter
*[RAT]: Register Allocation Table (Register Alias Table)
*[RAW]: Read-After-Write
*[SB]: Store Buffer
*[SW]: Store Word
*[WAR]: Write-After-Read
*[WAW]: Write-After-Write
*[RAR]: Read-After-Read