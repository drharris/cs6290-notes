---
id: ilp
title: ILP
sidebar_label: ILP
---

[🔗Lecture on Udacity](https://classroom.udacity.com/courses/ud007/lessons/3615429333/concepts/last-viewed)

## ILP All in the Same Cycle
Ideal scenario is all instructions executing during the same cycle. This may work sometimes for some instructions, but typically there will be dependencies that prevent this, as below.

![All instructions in same cycle](https://i.imgur.com/tQk5TyF.png)

## The Execute Stage

Can forwarding help? In the previous example, Inst 1 would be able to forward the result to Instruction 2 in the next cycle, but not during the same cycle. Instruction 2 would need to be stalled until the next cycle. But, if Instructions 3-5 do not have dependencies, they can continue executing during the current cycle.

## RAW Dependencies
Even the ideal processor that can execute any number of instructions per cycle still has to obey RAW dependencies, which creates delays. So, ideal CPI can never be 0. For example, for instructions 1, 2, 3, 4, and 5, where there is a dependency between 1-2, 3-3, and 4-5, it would take 3 cycles (cycle 1 executes 1 and 3, cycle 2 executes 2 and 4, cycle 3 executes 5), for a total CPI of 3/5 = 0.60



*[ALU]: Arithmetic Logic Unit
*[CPI]: Cycles Per Instruction
*[ILP]: Instruction-Level Parallelism
*[PC]: Program Counter
*[RAW]: Read-After-Write
*[WAR]: Write-After-Read
*[WAW]: Write-After-Write
*[RAR]: Read-After-Read