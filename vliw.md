---
id: vliw
title: VLIW
sidebar_label: VLIW
---

[🔗Lecture on Udacity (17 min)](https://classroom.udacity.com/courses/ud007/lessons/961349070/concepts/last-viewed)

## Superscalar vs VLIW

| | OOO Superscalar | In-Order Superscalar | VLIW |
| --- | --- | --- | --- |
| IPC | \\(\leq N\\) | \\(\leq N\\) | 1 large inst/cycle but does work of N "normal" inst |
| How to find independent instructions? | Look at >> N insts | Look at next N insts in program order | Just do next large inst |
| Hardware Cost | $$$ | $$ | $ |
| Help from compiler? | Compiler can help | Needs help | Completely depends on compiler for performance |

## The Good and The Bad

Good:
* Compiler does the hard work
  * Plenty of Time
* Simpler HW
* Can be energy efficient
* Works well on loops and "regular" code

Bad:
* Latencies not always the same
  * e.g. Cache Miss
* Irregular Applications
  * e.g. Applications with lots of decision making that are hard for compiler to figure out
* Code Bloat
  * Can be much larger if there are many no-ops from dependent instructions

## VLIW Instructions

* Instruction set typically has all the "normal" ISA opcodes
* Full predication support (or at least extensive)
  * Replies on compiler to expose parallelism via instruction scheduling
* Lots of registers
  * A lot of scheduling optimizations require use of additional registers
* Branch hints
  * Compiler tells processor what it thinks the branch will do
* VLIW instruction "compaction"
  | OP1 | NOP | NOP | NOP |
  |---|---|---|---|

  | OP2 | OP3 | NOP | NOP |
  |---|---|---|---|
  becomes
  | OP1 |X| OP2 | | OP3 |X| NOP | |
  |---|---|---|---|---|---|---|---|
  where the X represents some sort of stop bit. This reduces the number of `NOP` and therefore code bloat.

## VLIW Examples
* Itanium
  * _Tons_ of ISA Features
  * HW very complicated
  * Still not great on irregular code
* DSP Processors
  * Regular loops, lots of iterations
  * Excellent performance
  * Very energy-efficient

The main point is that VLIW can be a very good choice given the right application where compilers can do well.


*[ALU]: Arithmetic Logic Unit
*[CPI]: Cycles Per Instruction
*[DSP]: Digital Signal Processing
*[ILP]: Instruction-Level Parallelism
*[IPC]: Instructions per Cycle
*[IQ]: Instruction Queue
*[ISA]: Instruction Set Architecture
*[LB]: Load Buffer
*[LSQ]: Load-Store Queue
*[LW]: Load Word
*[OOO]: Out Of Order
*[PC]: Program Counter
*[RAT]: Register Allocation Table (Register Alias Table)
*[RAR]: Read-After-Read
*[RAW]: Read-After-Write
*[ROB]: ReOrder Buffer
*[SB]: Store Buffer
*[SW]: Store Word
*[VLIW]: Very Long Instruction Word
*[WAR]: Write-After-Read
*[WAW]: Write-After-Write
