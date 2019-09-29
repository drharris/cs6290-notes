---
id: compiler-ilp
title: Compiler ILP
sidebar_label: Compiler ILP
---

[🔗Lecture on Udacity (1 hr)](https://classroom.udacity.com/courses/ud007/lessons/972428795/concepts/last-viewed)


## Can compilers help improve IPC?

* Limited ILP
  * Due to dependence chains (each instruction depends on the one before it)
* Limited "Window" into program
  * Independent instructions are far apart

## Tree Height Reduction
Consider a program that performs the operation `R8 = R2 + R3 + R4 + R5`:
```mipsasm
    ADD R8, R2, R3
    ADD R8, R8, R4
    ADD R8, R8, R5
```
Obviously this creates a depdendence chain. Instead we could group the instructions like `R8 = (R2 + R3) + (R4 + R5)`, or:
```mipsasm
    ADD R8, R2, R3
    ADD R7, R4, R5
    ADD R8, R8, R7
```
This allows the first two instructions to execute in parallel. Tree Height Reduction in a compiler uses associativity to accomplish this. But, it should be considered that not all operations are associative.

## Make Indepdendent Instructions Easier to Find
Can use various techniques (to follow)
1. Instruction Scheduling (branch-free sequences of instructions)
2. Loop Unrolling (and how it interacts with Instruction Scheduling)
3. Trace Scheduling

## Instruction Scheduling
Different from Tomasulo's algorithm that takes place in the processor, but attempts to accomplish a similar thing. Take this sequence of instructions:

```mipsasm
Loop:
    LW   R2, 0(R1)
    ADD  R2, R2, R0
    SW   R2, 0(R1)
    ADDI R1, R1, 4
    BNE  R1, R3, Loop
```
On each cycle, it may look more like this:
```
1.  LW   R2, 0(R1)
2.  (stall)
3.  ADD  R2, R2, R0
4.  (stall)
5.  (stall)
6.  SW   R2, 0(R1)
7.  ADDI R1, R1, 4
8.  (stall)
9.  (stall)
10. BNE  R1, R3, Loop
```
Can we move something into that first stall to help ILP? Cycles 3 and 6 cannot move because those are dependent. Is it possible to move the ADDI from cycle 7 to cycle 2, since it does not depend on anything else?
```mipsasm
Loop:
    LW   R2, 0(R1)
    ADDI R1, R1, 4
    ADD  R2, R2, R0
    SW   R2, -4(R1)
    BNE  R1, R3, Loop
```
The `ADDI` can move as-is, but we need to correct the offset in the `SW` instruction to compensate for this. From a cycle analysis, we have eliminated cycles 7 (moved to 2), and the stall from 8-9 (the existing stalls in 4-5 will also compensate for the `ADDI` delay). So instead of 10 cycles, the loop now runs in 7 cycles.

## Scheduling and If-Conversion

In the following example, orange and green are two different branches, and using predication we attempt to execute both and throw away the wrong results later when the branch executes. So using If-Conversion, the program executes these in order. 

![Scheduling and If Conversion](https://i.imgur.com/eaW4xq6.png)

For the purposes of scheduling, we can always perform scheduling optimizations within each functional block (e.g. instructions inside the orange section), and also between consecutive blocks (white-orange). We can even reschedule instructions on either side of the branches, and maintain correctness.

## If-Convert a Loop
We know how to if-convert branches, but what about a loop?

```mipsasm
Loop:
    LW   R2, 0(R1)          #(stall cycle after)
    ADD  R2, R2, R3         #(stall cycle after)
    SW   R2, 0(R1)
    ADDI R1, R1, 4          #(stall cycle after)
    BNE  R1, R5, Loop
```
With scheduling, we get something like this (notice decrease in wasted cycles)
```mipsasm
Loop:
    LW   R2, 0(R1)
    ADDI R1, R1, 4
    ADD  R2, R2, R3         #(stall cycle after)
    SW   R2, -4(R1)
    BNE  R1, R5, Loop
```
Instead of the BNE, we could use something like If-Conversion to load in the next instructions and fill that last stall cycle. But each iteration would require a new predicate to be created, and there's a point at which most things only get done if a predicate is true, and we don't see the performance boost for the overhead that will take. So, we cannot really If-Convert, but we can do something called Loop Unrolling

## Loop Unrolling
```cpp
for(i=1000; i !=0;l i--) {
    a[i] = a[i] + s;
}
```
compiles to...
```mipsasm
    LW   R2, 0(R1)
    ADD  R2, R2, R3
    SW   R2, 0(R1)
    ADDI R1, R1, -4
    BNE  R1, R5, Loop
```

With loop unrolling, we try to do a few iterations of the loop during one iteration, maybe:
```cpp
for(i=1000; i !=0;l i=i-2) {
    a[i] = a[i] + s;
    a[i-1] = a[i-1] + s;   // we could try unrolling a few more times
}
```
which now compiles to:
```mipsasm
    LW   R2, 0(R1)
    ADD  R2, R2, R3
    SW   R2, 0(R1)
    LW   R2, -4(R1)
    ADD  R2, R2, R3
    SW   R2, -4(R1)
    ADDI R1, R1, -8
    BNE  R1, R5, Loop
```
So the process is to take the work, copy it twice, adjust offsets, then adjust the final loop counter and branch instructions as needed. This example was an "unroll once". Unrolling twice would perform 3 iterations before branching.

### Loop Unrolling Benefits: # Instructions \\(\downarrow \\)

In the example above, we start from 5 instructions * 1000 loops = 5000. After loop unrolling we have 8 instructions * 500 loops = 4000. From the Iron Law, we know Execution Time = (# inst)(CPI)(Cycle Time). So just by decreasing instructions by 20% there is a significant effect on overall performance.

###  Loop Unrolling Benefits: CPI \\(\downarrow \\)
Assume a processor with 4-Issue, In-Order, with perfect branch prediction. We can view how the iterations span over cycles:
|`Loop:`            | 1 | 2 | 3 | 4 | 5 | 6 |
|---                |---|---|---|---|---|---|
|`LW   R2, 0(R1)`   | x |   |   | x |   |   |
|`ADD  R2, R2, R3`  |   | x |   |   | x |   |
|`SW   R2, 0(R1)`   |   |   | x |   |   | x |
|`ADDI R1, R1, -4`  |   |   | x |   |   | x |
|`BNE  R1, R5, Loop`|   |   |   | x |   | ... |
For an overall CPI of 3/5. With scheduling, we can do the following:
|`Loop:`            | 1 | 2 | 3 | 4 | 5 | 6 |
|---                |---|---|---|---|---|---|
|`LW   R2, 0(R1)`   | x |   | x |   | x |   |
|`ADDI R1, R1, -4`  | x |   | x |   | x |   |
|`ADD  R2, R2, R3`  |   | x |   | x |   | x |
|`SW   R2, 4(R1)`   |   |   | x |   | x |   |
|`BNE  R1, R5, Loop`|   |   | x |   | x | ... |
For an overall CPI of 2/5 with scheduling. This is a significant boost. Now, what about loop unrolling (unrolled once)?
|`Loop:`            | 1 | 2 | 3 | 4 | 5 | 6 |
|---                |---|---|---|---|---|---|
|`LW   R2, 0(R1)`   | x |   |   |   |   | x |
|`ADD  R2, R2, R3`  |   | x |   |   |   |   |
|`SW   R2, 0(R1)`   |   |   | x |   |   |   |
|`LW   R2, -4(R1)`  |   |   | x |   |   |   |
|`ADD  R2, R2, R3`  |   |   |   | x |   |   |
|`SW   R2, -4(R1)`  |   |   |   |   | x |   |
|`ADDI R1, R1, -8`  |   |   |   |   | x |   |
|`BNE  R1, R5, Loop`|   |   |   |   |   | x |
So it takes 5 cycles to do 8 instructions, for a CPI of 5/8. This is slightly worse when only looking at a few iterations, but overall it will perform much better (since we need half the iterations). Finally... with unrolling once and scheduling:
|`Loop:`            | 1 | 2 | 3 | 4 |
|---                |---|---|---|---|
|`LW   R2, 0(R1)`   | x |   | x |   |
|`LW   R10, -4(R1)` | x |   |   | x |
|`ADD  R2, R2, R3`  |   | x |   |   |
|`ADD  R10, R10, R3`|   | x |   |   |
|`ADDI R1, R1, -8`  |   | x |   |   |
|`SW   R2, 8(R1)`   |   |   | x |   |
|`SW   R10, 4(R1)`  |   |   | x |   |
|`BNE  R1, R5, Loop`|   |   | x |   |
So it takes 3 cycles to perform 8 instructions, for CPI of 3/8. This is slightly better than with scheduling alone, and when the benefit of loop unrolling over time (half the iterations) are considered, it is a significant improvement. Unrolling provides more opportunities for scheduling to reorder things to optimize for parallelism, in addition to reducing the overall number of instructions.

### Unrolling Downside?
A few reasons we may not always unroll loops.

1. Code Bloat (in terms of lines of code after compilation)
2. What if number of iterations is unknown (e.g. while loop)?
3. What if number of iterations is not a multiple of N? (N = number of unrolls)

Solutions to 2 and 3 do exist, but are beyond the scope of this course (may be in a compilers course).

## Function Call Inlining

Function Call Inlining is an optimization that takes a function and copies the work inside the main program. 

![Function Call Inlining](https://i.imgur.com/Vl3YXY0.png)

This has the benefits of:
* Eliminating call/return overheads (reduces # instructions)
* Better scheduling (reduces CPI)
  * Without inlining, the compiler can only schedule instructions around and inside the function block, but by inlining it can schedule all instructions together.
* With fewer instructions and reduced CPI, execution time improves dramatically.

### Function Call Inlining Downside

The main downside to inlining, like in Loop Unrolling, is code bloat. Instead of abstracting the code into its own space and using call/return, it replicates the function body each time it is called. Each time it is inlined, it increases the total program size. Therefore we must be judicious about the usage of inlining. Ideally we select smaller functions to inline, and primarily those that result in fewer instructions when inlined compared to the overhead of setting up parameters, call, and return.

## Other IPC-Enhancing Compiler Stuff

* Software Pipelining
  * Treat a loop as a pipeline where you interleave the instructions among the cycles such that correctness is maintained but there are no dependencies)
* Trace Scheduling ("If-Conversion on steroids")
  * Combine the common path into one block of scheduled code, provide a way to branch and "fix" the wrong path execution if the uncommon path is determined to occur.

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