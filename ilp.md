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
Even the ideal processor that can execute any number of instructions per cycle still has to obey RAW dependencies, which creates delays and affects overall CPI. So, ideal CPI can never be 0. For example, for instructions 1, 2, 3, 4, and 5, where there is a dependency between 1-2, 3-4, and 4-5, it would take 3 cycles (cycle 1 executes 1 and 3, cycle 2 executes 2 and 4, cycle 3 executes 5), for a total CPI of 3/5 = 0.60. If every instruction had a dependency with the next, you can't do anything in parallel and the minimum CPI is 1.

## WAW Dependencies

In the below example, the second instructions has a data dependency on the first, and gets delayed one cycle. Meanwhile, all other cycles do not have any dependencies and can also be executed in the first cycle. However, we see in the last instruction that R4 could be written, but then due to the delay in instruction 2, could be overwritten. This out-of-order instruction could result in the final value of R4 not being what is expected based on the program. Thus the processor needs a way to find this dependency and delay the last instruction enough cycles to avoid the write issue.

![WAW Dependencies](https://i.imgur.com/q9akgJp.png)

## Removing False Dependencies

RAW dependencies are "true" dependencies - one instruction truly depends on data from another and the dependency must be obeyed. WAR and WAW dependencies are "false" (name) dependencies. They are named this because there is nothing fundamental about them - they are the result of using the same register to store results. If the second instruction used a different register value, there would be no dependency.

## Duplicating Register Values

In the case of a false dependency, you could simply duplicate registers by using separate versions of them. In the below example, you can store two versions of R4 - one on the 2nd instruction, and another on the 4th, and we remember both. The dependency can be resolved when the future instruction that uses R4 can "search" through those past versions and select the most recent.

![Duplicating Register Values](https://i.imgur.com/WtpuH48.png)

Likewise, instruction 3 must also search among "versions" of R4 in instructions 2 and 4 and determine the version it needs is from instruction 2. This is possible and correct, but keeping multiple version is very complicated.

## Register Renaming

Register renaming separates registers into two types:
- Architectural = registers that programmer/compiler use
- Physical = all places value can actually go within the processor

As the processor fetches and decodes instructions, it "rewrites" the program to use physical registers. This requires a table called the Register Allocation Table (RAT). This table says which physical register has a value for which architectural register.

### RAT Example
 ![RAT Example](https://i.imgur.com/foUlLDD.png)

## False Dependencies After Renaming?

In the below example, you can see the true dependencies in purple, and the output/anti dependencies in green. In our renamed program, only the true dependencies remain. This also results in a much lower CPI.

![False Dependencies After Renaming](https://i.imgur.com/LcbgTrG.png)

## Instruction Level Parallelism (ILP)

ILP is the IPC when:
- Processor does entire instruction in 1 cycle
- Processor can do any number of instructions in the same cycle
  - Has to obey True Dependencies

So, ILP is really what an ideal processor can do with a program, subject only to obeying true dependencies. ILP is a property of a ***program***, not of a processor.

Steps to get ILP:
1. Rename Registers - use RAT
2. "Execute" - ensure no false dependencies, determine when instructions are executed
3. Calculate ILP = (\# instructions)/(\# cycles)

### ILP Example

Tips:
1. You don't have to first do renaming, just pay attention to true dependencies, and trust renaming to handle false dependencies.
2. Be mindful to count how many cycles you're computing over
3. Make sure you're dividing the right direction (instructions/cycles)

![ILP Example](https://i.imgur.com/y1RdLrg.png)

## ILP with Structural and Control Dependencies

When computing ILP we only consider true dependencies, not false dependencies. But what about structural and control dependencies?

When considering ILP, there are no structural dependencies. Those are caused by lack of hardware parallelism. ILP assumes ideal hardware - any instructions that can possibly compute in one cycle will do so.

For control dependencies, we assume perfect same-cycle branch prediction (even predicted before it is executed). For example, below we see that the branch is predicted at the point of program load, such that the label instruction will execute in the first cycle.

|   | 1 | 2 | 3 |
|------------------|:---:|:---:|:---:|
| `ADD R1, R2, R3` | x | | |
| `MUL R1, R1, R1` | | x | |
| `BNE R5, R1, Label` | | | x |
| ... | | | |
| `Label:`<br />`MUL R5, R7, R8` | x | | |

## ILP vs IPC

ILP is not equal to IPC except on a perfect/ideal out-of-order processor. So IPC should be computed based on the properties of the processor that it is run on, as seen below (note: consider the IPC was calculated ignoring the "issue" property).

![ILP vs IPC](https://i.imgur.com/m8vSTGJ.png)

Therefore, we can state ILP \\(\geq\\) IPC, as ILP is calculated using no processor limitations.

### ILP and IPC Discussion

The following are considerations when thinking about effect of processor issue width and order to maximize IPC.

![ILP and IPC discussion](https://i.imgur.com/F5utKaZ.png)


*[ALU]: Arithmetic Logic Unit
*[CPI]: Cycles Per Instruction
*[ILP]: Instruction-Level Parallelism
*[IPC]: Instructions per Cycle
*[PC]: Program Counter
*[RAT]: Register Allocation Table
*[RAW]: Read-After-Write
*[WAR]: Write-After-Read
*[WAW]: Write-After-Write
*[RAR]: Read-After-Read