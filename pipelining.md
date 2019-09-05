---
id: pipelining
title: Pipelining
sidebar_label: Pipelining
---

[🔗Lecture on Udacity](https://classroom.udacity.com/courses/ud007/lessons/3650589023/concepts/last-viewed)

Given a certain latency, the idea is that the first unit of work is achieved in the same period of time, but successive units continue to be completed thereafter. (Oil pipeline vs. traveling with a bucket)

## Pipelining in a Processor
Each logical phase of operation in the processor (fetch, decode, ALU, memory, write) operates as a pipeline in which successive instructions move through each phase immediately behind the current instruction, instead of waiting for the entire instruction to complete.

![Pipelining in a Processor](https://i.imgur.com/0m5vXEf.png)

## Pipeline CPI
Is Pipeline CPI = always 1?

On initial fill, CPI \\( \rightarrow \\) 1 when # instruction \\( \rightarrow \infty \\). If pipeline stalls (a single phase breaks down), then on the next cycle the successive phase will be doing no work. Therefore, CPI is always > 1 if any stalls are possible. For example, if every 5 cycles one phase stalls, your CPI is 6 cycles/5 units = 1.2 CPI.

## Processor Pipeline Stalls

In the example below, we see one example of a pipeline stall in a processor in a program like this:

```mipsasm
LW  R1, ...
ADD R2, R1, 1
ADD R3, R2, 1
```

![Processor Pipeline Stalls](https://i.imgur.com/E21sE3o.png)

The second instruction depends on the result of the first instruction, and must wait for it to complete the ALU and MEM phases before it can proceed. Thus, the CPI is actually much greater than 1.

## Processor Pipeline Stalls and Flushes

```mipsasm
LW  R1, ...
ADD R2, R1, 1
ADD R3, R2, 1
JUMP ...
SUB ... ...
ADD ... ...
SHIFT
```

In this case, we have a jump, but we don't know where yet (maybe the address is being manipulated by previous instructions). Instructions following the jump instruction are loaded into the pipeline. When we get to the ALU phase, the `JUMP` will be processed, but this means the next instruction is not the `SUB ... ...` or `ADD ... ...`. These instructions are then flushed from the pipeline and the correct instruction (`SHIFT`) is fetched.

![Processor Pipeline Stalls and Flushes](https://i.imgur.com/5S9f2kB.png)

## Control Dependencies

For the following program, the `ADD`/`SUB` instructions have a control dependence on `BEQ`. Similarly, the instructions after `label:` also have a control dependence on `BEQ`. 

```mipsasm
    ADD R1, R1, R2
    BEQ R1, R3, label
    ADD R2, R3, R4
    SUB R5, R6, R8
label:
    MUL R5, R6, R8
```

We estimate that:
- 20% of instructions are branch/jump
- slightly more than 50% of all branch/jump instructions are taken

On a 5-stage CPI, CPI = 1 + 0.1*2 = 1.2 (10% of the time (50% * 20%) an instruction spends two extra cycles)

With a deeper pipeline (more stages), the number of wasted instructions increases.

## Data Dependencies

```mipsasm
    ADD R1, R2, R3
    SUB R7, R1, R8
    MUL R1, R5, R6
```

This program has 3 dependencies:

1. Lines 1 and 2 have a RAW (Read-After-Write) dependence on R1 (also called Flow dependence, or TRUE dependence). The `ADD` instruction must be completed before the `SUB` instruction.

2. Lines 1 and 3 have a WAW (Write-After-Write) dependence on R1 with the `MUL` instruction in which the `ADD` must complete first, else R1 is overwritten with an incorrect value. This is also called an Output dependence.

3. Lines 2 and 3 have a WAR (Write-After-Read) dependence on R1 in that the `SUB` instruction must use the value of R1 before the `MUL` instruction overwrites it. This is also called an Anti-dependence because it reverses the order of the flow dependence.

WAW and WAR dependencies are also called "False" or "Name" dependencies. RAR (Read-After-Read) dependencies do not matter since the value could not have changed in-between and is thus safe to read.

### Data Dependencies Quiz

(Included for additional study help). For the following program, select which dependencies exist:

```mipsasm
    MUL R1, R2, R3
    ADD R4, R4, R1
    MUL R1, R5, R6
    SUB R4, R4, R1
```

|   | RAW | WAR | WAW |
|---|---|---|---|
| \\( I1 \rightarrow I2 \\) | x | - | - |
| \\( I1 \rightarrow I3 \\) | - | - | x |
| \\( I1 \rightarrow I4 \\) | - | - | - |
| \\( I2 \rightarrow I3 \\) | - | x | - |

## Dependencies and Hazards

Dependence - property of the program alone.

Hazard - when a dependence results in incorrect execution.

For example, in a 5-stage pipeline, a dependency that is 3 instructions apart may not cause a hazard, since the result will be written before the dependent instruction reads it. 

## Handling of Hazards

First, detect hazard situations. Then, address it by:
1. Flush dependent instructions
2. Stall dependent instruction
3. Fix values read by dependent instructions

Must use flushes for control dependencies, because the instructions that come after the hazard are the wrong instructions.

For data dependence, we can stall the next instruction, or fix the instruction by forwarding the value to the correct stage of the pipeline (e.g. "keep" the value inside the ALU stage for the next instruction to use). Forwarding does not always work, because the value we need is produced at a later point in time. In this cases we must stall.

## How Many Stages?

For an ideal CPI = 1, we consider the following:

More Stages \\( \rightarrow \\) more hazards (CPI \\( \uparrow \\)), but less work per stage ( cycle time \\( \downarrow \\))

From iron law, Execution Time = # Instructions * CPI * Cycle Time

\# Stages should be chosen to balance CPI and Cycle time (some local minima for execution time where cycle time has decreased without causing additional hazards). Additionally consider more stages consumes more power (work being done in less cycle time with more latches per stage). 

* Performance (execution time) \\( \Rightarrow \\) 30-40 stages
* Manageable Power Consumption \\( \Rightarrow \\) 10-15 stages

*[ALU]: Arithmetic Logic Unit
*[CPI]: Cycles Per Instruction
*[PC]: Program Counter
*[RAW]: Read-After-Write
*[WAR]: Write-After-Read
*[WAW]: Write-After-Write
*[RAR]: Read-After-Read