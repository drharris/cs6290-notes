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