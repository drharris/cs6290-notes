---
id: branches
title: Branches
sidebar_label: Branches
---

[🔗Lecture on Udacity](https://classroom.udacity.com/courses/ud007/lessons/3618489075/concepts/last-viewed)

## Branch in a Pipeline

```mipsasm
BEQ R1, R2, Label
```

A branch instruction like this compares R1 and R2, and if equal jumps to `Label` (usually by having in the immediate part of the instruction field the difference in PC for the next instruction versus the labeled instruction, such that it can simply add this offset when branched).

In a 5-stage pipeline, the compare happens in the third (ALU) stage. Meanwhile, two instructions have moved into Fetch/Read stages. If we fetched the correct instructions for the branch (or not), they continue to move correctly with no penalty. Otherwise we must flush and take a 2-instruction penalty.

Thus, it never pays to simply not fetch something as you always take the penalty, and it is better to take a penalty only sometimes than all the time. Another important note is that when fetching the instruction after `BEQ`, we know nothing about the branch itself yet except its address (we don't even know it's a branch at all yet), but we already must make a prediction of whether it's a taken branch or not.

## Branch Prediction Requirements

- Using only the knowledge of the instruction's PC
  - Guess PC of next instruction to fetch
- Must correctly guess: 
  - is this a branch?
  - is it taken?
  - if taken, what is the target PC?

The first two guesses can be combined into "is this a taken branch?"

## Branch Prediction Accuracy

$$ CPI = 1 + \frac{mispred}{inst} * \frac{penalty}{mispred} $$

The \\( \frac{mispred}{inst} \\) part is determined by the predictor accuracy. The \\(\frac{penalty}{mispred} \\) part is determined by the pipeline (where in the pipeline we figure out the misprediction).

Assumption below: 20% of all instructions are branches (common in programs).

| Accuracy \\(\downarrow\\) | Resolve in 3rd stage | Resolve in 10th stage |
|---|:---:|:---:|
| 50% for BR<br>100% all other | \\(1 + 0.5\*0.2\*2\\)<br>\\(= 1.2\\) | \\(1 + 0.5\*0.2\*9\\)<br>\\(= 1.9\\) |
| 90% of BR<br>100% all other | \\(1 + 0.1\*0.2\*2\\)<br>\\(= 1.04\\) | \\(1 + 0.1\*0.2\*9\\)<br>\\(= 1.18\\) |
| _(Speedup)_ | _1.15_ | _1.61_ |

Conclusions: A better branch predictor will help regardless of the pipeline, but the _amount_ of help changes with the pipeline depth.

## Performance with Not-Taken Prediction

| | Refuse to Predict<br>(stall until sure) | Predict Not-Taken<br>(always increment PC) |
|---|---|---|
| Branch | 3 cycles | 1 or 3 cycles |
| Non-Branch | 2 Cycles | 1 cycle |

Thus, Predict Not-Taken always wins over refusing to predict

## Predict Not-Taken

Operation: Simply increment PC (no special hardware or memory, since we have to do this anyway)

Accuracy:
* 20% of instructions are branches
* 60% of branches are taken
* \\(\Rightarrow\\) Correctness: 80% (non-branches) + 8% (non-taken branches)
* \\(\Rightarrow\\) Incorrect 12% of time
* CPI = 1 + 0.12*penalty

## Why We Need Better Prediction?
| | Not Taken<br>88% | Better<br>99% | Speedup |
|---|:---:|:---:|:---:|
| 5-stages<br>(3rd stage) | \\(1 + 0.12\*2\\)<br>\\(CPI = 1.24\\) | \\(1 + 0.01\*2\\)<br>\\(CPI = 1.02\\) | \\(1.22\\) |
| 14-stages<br>(11th stage) | \\(1 + 0.12\*10\\)<br>\\(CPI = 2.2\\) | \\(1 + 0.01\*10\\)<br>\\(CPI = 1.1\\) | \\(2\\) |
| 11th stage<br>(4 inst/cycle) | \\(0.25 + 0.12\*10\\)<br>\\(CPI = 1.45\\) | \\(0.25 + 0.01\*10\\)<br>\\(CPI = 0.35\\) | \\(4.14\\) |

If we have a deeper pipeline or are able to execute more instructions per cycle, the better predictor is more important than in simpler processors, because the cost of misprediction is much higher (more instructions lost with a misprediction).

## Better Prediction - How?

Predictor must compute \\(PC_{next}\\) based only on knowledge of \\(PC_{now}\\). This is not much information to decide on. It would help if we knew: 
* Is it a branch?
* Will it be taken?
* What is the offset field of the instruction?

But we don't know any of these because we're still fetching the instruction. We do, however, know the history of how \\(PC_{now}\\) has behaved in the past. So, we can go from: \\(PC_{next} = f(PC_{now})\\) to:

$$ PC_{next} = f(PC_{now}, history[PC_{now}]) $$

## BTB - Branch Target Buffer

The predictor can take the \\(PC_{now}\\) and uses it to index into a table called the BTB, with the output of our best guess at next PC. Later, when the branch executes, we know the actual \\(PC_{next}\\) and can compare with the predicted one. If it doesn't match, then it is handled as a misprediction and the BTB can be updated.

![Branch Target Buffer](https://i.imgur.com/h6Fwke1.png)

One problem: How big does the BTB need to be? We want it to have single cycle latency (small). However, it needs to contain an entire 64-bit address and we one entry for each PC we can fetch. Thus, the BTB would need to be as large as the program itself. How do we make it smaller?

## Realistic BTB

First, we don't need an entry for every possible PC. It's enough if we have entries for any PC likely to execute soon. For example, in a loop of 100 instructions, it's enough to have about a 100-length BTB.

Perhaps we find through testing that a 1024-entry BTB can still be accessed in one cycle. How do we map 64-bit PCs to this 1024 entry table, keeping in mind any delay in calculating the mapping from PC to BTB index would then shorten the BTB further to compensate.

The way we do this is by simply taking the last 10 bits (in the 1024 case). While this means future instructions will eventually overwrite the BTB entries for the current instructions, this ensures instructions located around each other are all mapping to the BTB during execution. This particularly applies to better predicting branch behavior in loops or smaller programs.