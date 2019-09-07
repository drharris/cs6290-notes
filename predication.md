---
id: predication
title: Predication
sidebar_label: Predication
---

[🔗Lecture on Udacity](https://classroom.udacity.com/courses/ud007/lessons/3617709440/concepts/last-viewed)

## Predication
Predication is another way we can try to deal with control hazards and dependencies by dividing the work into both directions.

Branch prediction
- Guess where it is going
- No penalty if correct
- Huge penalty if wrong (misprediction cost ~50 instructions)

Predication
- Do work of both directions
- Waste up to 50%
- Throw away work from wrong path

| Construct/Type | Branch Prediction | Predication | Winner |
| --- | --- | --- | --- |
| Loop | Very good, tends to take same path more often | Extra work is outside the loop and often wasted | Predict |
| Call/Ret | Can be perfect, always takes this path | Any other branch is always wasted | Predict|
| Large If/Then/Else | May waste large number of instructions | Either way we lose a large number of instructions | Predict |
| Small If/Then/Else | May waste a large number of instructions | Only waste a few instructions | Predication, depending on BPred accuracy |

## If-Conversion
Compiler takes code like this:
```cpp
if(cond) {
    x = arr[i];
    y = y+1;
} else {
    x = arr[j];
    y = y-1;
}
```
And changes it to this (work of both paths):
```cpp
x1 = arr[i];
x2 = arr[j];
y1 = y+1;
y2 = y-1;
x = cond ? x1 : x2;
y = cond ? y1 : y2;
```

There is still a question of how to do the conditional assignment. For example if we do something like this:

```mipsasm
BEQ ...
MOV X, X2
B Done
MOV X, X1
... etc.
```
then we haven't done much, because we just converted one branch into another branch and possibly have two mispredictions. We need a move instruction that is conditional on some flag being set.

## Conditional Move
In MIPS instruction set there are the following instructions

|inst | operands | does |
|---|---|---|
| `MOVZ` | Rd, Rs, Rt | `if(Rt == 0) Rd=Rs;` |
| `MOVN` | Rd, Rs, Rt | `if(Rt != 0) Rd=Rs;` |

So we can implement our conditional assignment `x = cond ? x1 : x2;`  as follows:

```mipsasm
R3 = cond
R1 = ... x1 ...
R2 = ... x2 ...
MOVN X, R1, R3
MOVZ X, R2, R3
```

Similarly in x86 there are many instructions (`CMOVZ`, `CMOVNZ`, `CMOVGT`, etc.) that operate based on the flags, where the move operation completes based on the condition codes provided.


*[2BP]: 2-bit Predictor
*[2BC]: 2-Bit Counter
*[2BCs]: 2-Bit Counters
*[ALU]: Arithmetic Logic Unit
*[BHT]: Branch History Table
*[BTB]: Branch Target Buffer
*[CPI]: Cycles Per Instruction
*[IF]: Instruction Fetch
*[PC]: Program Counter
*[PHT]: Pattern History Table
*[RAS]: Return Address Stack
*[XOR]: Exclusive-OR
