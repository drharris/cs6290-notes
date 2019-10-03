---
id: cheat-sheet-midterm
title: Midterm Cheat Sheet
sidebar_label: Cheat Sheet - Midterm
---

A denser collection of important pieces of information covering lectures from Introduction to VLIW

## Power Consumption Types

Two kinds of power a processor consumes.
1. Dynamic Power - consumed by activity in a circuit
   Computed by \\(P = \tfrac 12 \*C\*V^2\*f\*\alpha\\), where
    \\(C = \text{capacitance}\\), \\(P = \text{power supply voltage}\\), \\(f = \text{frequency}\\), \\(\alpha = \text{activity factor (% of transistors active each clock cycle)}\\)
2. Static Power - consumed when powered on, but idle
   - The power it takes to maintain the circuits when not in use.
   - V \\(\downarrow\\), leakage \\(\uparrow\\)

## Performance

* Latency (time start \\( \rightarrow \\) done)
* Throughput (#/second) (not necessarily 1/latency due to pipelining)
* Speedup - "X is N times faster than Y" (X new, Y old)
  * Speedup = speed(X)/speed(Y)
  * Speedup = throughput(X)/throughput(Y) = IPC(X)/IPC(Y)
  * Speedup = latency(Y)/latency(X) = \\(\frac{CPI(Y)\*CTime(Y)}{CPI(X)\*CTime(X)}\\) (notice Y/X reversal)
    * Can also multiply by nInst(Y)/nInst(X) factor
* Performance ~ Throughput ~ 1/Latency
* Ratios (e.g. speedup) can only be calculated via geometric mean
  * \\(\text{geometric mean} = \sqrt[n]{a_1\*a_2\*...a_n}\\)
* Iron Law of Performance:
  * **CPU Time** = (# instructions in the program) * (cycles per instruction) * (clock cycle time)
    * clock cycle time = 1/freq
  * For unequal instruction times: \\(\sum_i (IC_i\* CPI_i) * \frac{\text{time}}{\text{cycle}}\\)
* Amdahl's Law - overall effect due to partial change
  * \\(speedup = [(1-frac_{enh}) + \frac{frac_{enh}}{speedup_{enh}}]^{-1}\\)
  * \\( frac_{enh} \\) represents the fraction of the execution **TIME**
  * Consider diminishing returns by improving the same area of code.

## Pipelining
* CPI = (base CPI) + (stall/mispred rate %)*(mispred/stall penalty in cycles)
* Dependence - property of the program alone
  * Control dependence - one instruction executes based on result of previous instruction
  * Data Dependence
    * RAW (Read-After-Write) - "Flow" or "True" Dependence
    * WAW (Write-After-Write) - "Output", "False", or "Name" dependence
    * WAR (Write-After-Read) - "Anti-", "False", or "Name" dependence
* Hazard - when a dependence results in incorrect execution
  * Handled by Flush, Stall, and Fix values
* More Stages \\( \rightarrow \\) more hazards (CPI \\( \uparrow \\)), but less work per stage ( cycle time \\( \downarrow \\))
* 5-stage pipeline: Fetch-Decode-Execute-Memory-Write

## Branch Prediction
* Predictor must compute \\(PC_{next}\\) based only on knowledge of \\(PC_{now}\\)
  1. Guess "is this a branch?"
  2. Guess "is it taken?"
  3. "and if so, what is the target PC"
* Accuracy: \\( CPI = 1 + \frac{mispred}{inst} * \frac{penalty}{mispred} \\)
  * \\( \frac{mispred}{inst} \\) is determined by the predictor accuracy.
  * \\(\frac{penalty}{mispred} \\) is determined by the pipeline depth at misprediction.
* Types of Predictors and components
  * Not-Taken: Always assume branch is not taken
  * Historical: \\(PC_{next} = f(PC_{now}, history[PC_{now}])\\)
  * Branch Target Buffer (BTB): simple table of next best guessed PC based on current PC
    * Use last N bits of PC (not counting final 2-4 alignment bits) to index into this table
  * Branch History Table (BHT): like BTB, but entry is a single bit that tells us Taken/Not-Taken
    * 1-bit history does not handle switches in behavior well
    * 2-bit history adds hysteresis (strong and weak taken or not-taken)
    * 3-bit, 4-bit? Depends on pattern of anomalies, but may not be worth it.
  * History-Based Predictor: Looks at last N states of taken/not-taken.
    * Example: in sequence NNTNNT, if history is NN we know to predict T.
    * 1-bit history with 2-bit counters (1 historical state, 2-bit prediction)
    * 2-bit history with 2-bit counters (2 historical states, 2-bit prediction)
    * Shared counters - Pattern History Table (PC-indexed N bits per entry, XOR to index into BHT with 2-bit counter entries)
      * PShare - "private history, shared counters" - inner loops, smaller patterns
      * GShare - "global history, shared counters" - correlated branches across program
  * Tournament Predictor
    * Meta-Predictor feeds into a PShare and GShare, and is trained based on the results of each.
  * Hierarchical Predictor
    * Uses one good and one ok predictor (use "ok" predictor for easy branches, and "good" predictor for difficult ones)
* Return Address Stack (RAS) - dedicated to predicting function returns
  * Should be very small structure, quick, and accurate (fairly deterministic)
  * Can still mispredict - wrap around stack due to limited space.
  * Can know an instruction is a `RET` via 1BC or pre-decoding instructions

## Predication
Attempts to do the work of both directions of a branch and simply waste the work if wrong (to avoid control hazards)
* If-Conversion (takes normal code and makes it perform both paths)
    ```cpp
    if(cond) {           |>|   x1 = arr[i];
        x = arr[i];      |>|   x2 = arr[j];
        y = y+1;         |>|   y1 = y+1;
    } else {             |>|   y2 = y-1;
        x = arr[j];      |>|   x = cond ? x1 : x2;
        y = y-1; }       |>|   y = cond ? y1 : y2;
    ```
* MIPS operands to help with this (and what `x = cond ? x1 : x2;` looks like)
    |inst | operands | does |
    |---|---|---|
    | `MOVZ` | Rd, Rs, Rt | `if(Rt == 0) Rd=Rs;` |
    | `MOVN` | Rd, Rs, Rt | `if(Rt != 0) Rd=Rs;` |
    ```mipsasm
    R3 = cond
    R1 = ... x1 ...
    R2 = ... x2 ...
    MOVN X, R1, R3
    MOVZ X, R2, R3
    ```
* If-Conversion takes more instructions to do the work, but avoids any penalty, so is typically more performant.

## Instruction Level Parallelism (ILP)
ILP is the IPC when the processor does the entire instruction in 1 cycle, and can do any number of instructions in the same cycle (while obeying true dependencies)
* Register Allocation Table (RAT) is used for renaming registers
* Steps to get ILP value:
  1. Rename Registers - use RAT
  2. "Execute" - ensure no false dependencies, determine when instructions are executed
  3. Calculate ILP = (\# instructions)/(\# cycles)
     1. Pay attention to true dependencies, trust renaming to handle false dependencies.
     2. Be mindful to count how many cycles being computed over
     3. Assume ideal hardware - all instructions that can compute, will.
     4. Assume perfect same-cycle branch prediction
* IPC should never assume "perfect processor", so ILP \\(\geq\\) IPC.

## Instruction Scheduling (Tomasulo)
![Tomasulo's Algorithm](https://i.imgur.com/MuCQEgr.png)
All of these things happen every cycle:
1. Issue
   Take next from IQ, determine inputs, get free RS and enqueue, tag destination reg of instruction
2. Dispatch
   As RAT values become available on result bus, move from RS to Execution
3. Write Result (Broadcast)
   When execution is complete, put tag and result on bus, write to RF, update RAT, free RS

## ReOrder Buffer (ROB)
Used to prevent issues with exceptions and mispredictions to ensure results are not committed to the actual register before previous instructions have completed.

Correct Out-Of-Order Execution
* Execute Out-Of-Order
* Broadcast Out-Of-Order
* Write values to registers In-Order!

ROB is a structure (Register | Value | Done) that sits between the RAT and RS. RS now only dispatches instructions and does not have to wait for result to be broadcast before freeing a spot in the RS). RF is only written to once that instruction is complete and all previous instructions have been written. ROB ensures no wrong registers have been committed; upon exception it can flush and move to exception handler.

## Memory Ordering
Uses Load-Store Queue (LSQ) to handle read/write memory dependencies. This allows forwarding from stores to loads without having to access cache/mem. Instructions may be out of order, but all memory accesses are in-order. LSQ considers the followiung when being used:
1. `LOAD`: Which earlier `STORE` can I get a value from?
2. `STORE`: Which later `LOAD`s do I need to give my value to?

- Issue: Need a ROB entry and an LSQ entry
- Execute Load/Store: compute the address, produce the value (simultaneously)
- (`LOAD` only) Write Result and Broadcast it
- Commit Load/Store: Free ROB & LSQ entries
  - (`STORE` only) Send write to memory

## Compiler ILP
* Tree Height Reduction uses associativity in operations to avoid chaining dependencies (e.g. x+y+z+a becomes (x+y)+(z+a)).
* Instruction Scheduling attempts to reorder instructions to fill in any natural "stalls"
* Loop Unrolling performs multiple iterations of the loop during one actual branching. (unroll once means 2 iterations before branch).
* Function Call Inlining takes a function and copies the work to the main program, providing opportunities for scheduling or eliminating call/ret.

## VLIW
Combines multiple instructions into one large one - requires extensive compiler support, but lowers hardware cost.