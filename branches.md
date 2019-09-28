---
id: branches
title: Branches
sidebar_label: Branches
---

[🔗Lecture on Udacity (2.5 hr)](https://classroom.udacity.com/courses/ud007/lessons/3618489075/concepts/last-viewed)

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

### Branch Prediction Accuracy

$$ CPI = 1 + \frac{mispred}{inst} * \frac{penalty}{mispred} $$

The \\( \frac{mispred}{inst} \\) part is determined by the predictor accuracy. The \\(\frac{penalty}{mispred} \\) part is determined by the pipeline (where in the pipeline we figure out the misprediction).

Assumption below: 20% of all instructions are branches (common in programs).

| Accuracy \\(\downarrow\\) | Resolve in 3rd stage | Resolve in 10th stage |
|---|:---:|:---:|
| 50% for BR<br>100% all other | \\(1 + 0.5\*0.2\*2\\)<br>\\(= 1.2\\) | \\(1 + 0.5\*0.2\*9\\)<br>\\(= 1.9\\) |
| 90% of BR<br>100% all other | \\(1 + 0.1\*0.2\*2\\)<br>\\(= 1.04\\) | \\(1 + 0.1\*0.2\*9\\)<br>\\(= 1.18\\) |
| _(Speedup)_ | _1.15_ | _1.61_ |

Conclusions: A better branch predictor will help regardless of the pipeline, but the _amount_ of help changes with the pipeline depth.

### Performance with Not-Taken Prediction

| | Refuse to Predict<br>(stall until sure) | Predict Not-Taken<br>(always increment PC) |
|---|---|---|
| Branch | 3 cycles | 1 or 3 cycles |
| Non-Branch | 2 Cycles | 1 cycle |

Thus, Predict Not-Taken always wins over refusing to predict

### Predict Not-Taken

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

### Better Prediction - How?

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

### Realistic BTB

First, we don't need an entry for every possible PC. It's enough if we have entries for any PC likely to execute soon. For example, in a loop of 100 instructions, it's enough to have about a 100-length BTB.

Perhaps we find through testing that a 1024-entry BTB can still be accessed in one cycle. How do we map 64-bit PCs to this 1024 entry table, keeping in mind any delay in calculating the mapping from PC to BTB index would then shorten the BTB further to compensate.

The way we do this is by simply taking the last 10 bits (in the 1024 case). While this means future instructions will eventually overwrite the BTB entries for the current instructions, this ensures instructions located around each other are all mapping to the BTB during execution. This particularly applies to better predicting branch behavior in loops or smaller programs.

If instructions are word-aligned, the last two bits will always be `0b00`. Therefore we can ignore those and index using bits 12-2 in the 1024 case.

## Direction Predictor
The BHT is used like the BTB, but the entry is a single bit that tells us whether a PC is:
- [0] not a taken branch (PC++)
- [1] a taken branch (use BTB)

The entries are accessed by least insignificant bits of PC (e.g. 12-2 in 1024 case). Once the branch resolves we can update BHT accordingly. Because this table is very small in terms of data, it can be much larger in terms of entries.

### Problems with 1-bit Predictor
It works well if an instruction is always taken or else always not taken. It also predicts well when taken branches >>> not taken, or not taken >>> taken. Every "switch" (anomaly) between an instruction being taken and not taken causes two mispredictions (on the change, and then the next change).

So, the 1-bit predictor does not do so well if the taken to not taken ratio is not very large. It also will not perform well in short because this same anomaly occurs when the loop is executed again (the previous loop exit will cause it to mispredict the new loop).

## 2-Bit Predictor (2BP, 2BC)
This predictor fixes the behavior of the 1-bit predictor during the anomaly. The upper bit behaves like the 1-bit predictor, but it adds a hysteresis (or "conviction") bit.

![1-bit predictor state machine](https://i.imgur.com/pe7RYc9.png)

| Prediction Bit | Hysteresis Bit | Description |
|:---:|:---:|---|
| 0 | 0 | Strong Not-Taken |
| 0 | 1 | Weak Not-Taken |
| 1 | 0 | Weak Taken |
| 1 | 1 | Strong Taken | 

Basically, the upper bit controls the final prediction, but the lower bit allows the predictor to flow towards the opposite prediction. Thus a `0b00` state would require 2 taken branches in a row to change its prediction toward a taken branch. This prevents the case of a single anomaly causing multiple mispredictions, in that the behavior itself must be changing for the prediction to change.

![2-bit predictor state machine](https://i.imgur.com/TbSV6zv.png)

It can be called a 2-bit counter because it simply increments on taken branches, or decrements on not-taken. This allow the predictor to be easily implementable.

### 2-Bit Predictor Initialization
The question is - where do we start with the predictor? If we start in a strong state and are correct, then we have no mispredictions. If we are wrong, it costs two mispredictions before it corrects itself. If we start in a weak state and are right, we also have perfect prediction. However, if we were wrong, it only costs a single misprediction before it corrects itself. This leads us to think it's always best to start in a weak state.

However, consider the state where a branch flips between Taken and Not-Taken; by starting in a strong state, we mispredict half the time. Starting in a weak state, we _constantly_ mispredict.

![2-bit predictor misprediction on flipped state](https://i.imgur.com/LcNJv41.png)

So, while it may seem better to start in one state or another, in reality there is no way to predict what state is best for the incoming program, and indeed there is always some sequence of taken/not-taken that can result in constant misprediction. Therefore, it is best to simply initialize in the easiest state to start with, typically `0b00`.

### 1-bit to 2-bit is Good... 3-bit, 4-bit?

If there is benefit in moving to 2-bit, what about just adding more bits? This really serves to increase the cost (larger BHT to serve all PCs), but is only good when anomalous outcomes happen in streaks. How often does this happen? Sometimes. Maybe 3 bits might be worth it, but likely not 4. Best to stick with 2BP.

## History-Based Predictors
These predictors function best with a repeatable pattern (e.g. "N-T-N-T-N-T..." or "N-N-T-N-N-T..."). These are 100% predictable, just not with simple n-bit counters. So, how do we learn the pattern?

![history-based predictor](https://i.imgur.com/QFxW4dB.png)

In this case our prediction is done by looking "back" at previous steps. So in the first pattern, we know when the history is N, predict T, and vice-versa. In the second pattern, we look back two steps. So when the history is NN, predict T. When it is NT, predict N. And when it is TN, predict N.

### 1-bit History with 2-bit Counters
In this predictor, each BHT entry has a single history bit, and two 2-bit counters. The history bit can then be used to index into which counter to use for the prediction.

![1-bit history with 2-bit counters](https://i.imgur.com/wn9dPMD.png)

While this type of predictor works great for patterns like this, it still mispredicts 1/3 of the time in a "NNT-NNT-NNT" type pattern, as the prediction following an N is a 50% chance of being right.

### 2-bit History Predictor
This predictor works the same way as the 1-bit history predictor, but now we have 2 bits of history used to index into a 2BC[4] array. This perfectly predicts both the (NT)* and (NNT)* pattern types and is a pretty good predictor for other patterns. However, it wastes one 2BC on the (NNT)* case and two 2BCs on the (NT)* case.

![2-bit history predictor](https://i.imgur.com/7WaZWRg.png)

### N-bit History Predictor

We can generalize to state that an N-bit history predictor can successfully predict all taken patterns of \\(length \leq N+1\\), but will cost \\(N+2*2^N\\) bits per entry and waste most 2BCs. So, while increasing N will give us the ability to predict longer patterns, we do so at rapidly increasing cost with more waste.

## History-Based Predictors with Shared Counters

Instead of \\(2^N\\) counters per entry, we want to use \\(\approx N\\) counters. The idea is to share 2BCs between entries instead of each entry having its own counter.

We can do this with a Pattern History Table (PHT). This table simply keeps some PC-indexed history bits (N bits per entry), combines that with bits of the PC (XOR) to index into the BHT, each entry of which is just a single 2BC. Thus it is very possible to have two entries/history combinations using the same BHT entry

![history with shared counters](https://i.imgur.com/7PeYo5u.png)

Thus, small patterns will only use a few counters, leaving many other counters for longer, more complex patterns to use. This can still result in wasted space, but not nearly as much as the exponential increase of the N-bit history predictor. The downside, of course, is that some branches with particular histories may overlap with other branches/histories. But, if the BHT is large enough (and it can be with each entry being a single 2BC), this should happen rarely.

### PShare and GShare Predictors
This shared counters predictor is called PShare -> "P"rivate history, "Share"d counters. This is good for even-odd and 8-iteration loops.

Another option is GShare, or "G"lobal history, "Share"d counters. The history indexes are shared among all entries, and the PC+History operation (XOR) is used to index into the BHT. This is good for correlated branches - which is very common in programs, as typically operations in one branch are probably somewhat dependent on operations from a previous branch.

Which to use? Both! Use GShare for correlated branches, and PShare for single branches with shorter history.

## Tournament Predictor
We have two predictors, PShare and GShare, each of which is better at predicting certain types of branches. A meta-predictor (array of 2BCs) is used not as a branch predictor, but rather as a predictor of which other predictor is more likely to yield a correct prediction for the current branch. At each step, you "train" each individual predictor based on the outcome, and you also train the meta-predictor on how well each predictor is doing. 

![tournament predictor](https://i.imgur.com/Pa56hP0.png)

## Hierarchical Predictor
Like a tournament predictor, but instead of combining two good predictors, it is using one good and one ok predictor. The idea is that good predictors are expensive, and some branches are very easy to predict. So the "ok" predictor can be used for these branches, and the better predictor can be saved for the branches that are more difficult to predict.

| | Tournament | Hierarchical |
|---|---|---|
|Predictors|2 good predictors|1 good, 1 ok|
|Updates|Update both for every decision|Update OK-pred on every decision<br />Update Good-pred only if OK-pred not good|
|(Other)|Good predictors are both chosen to be balanced between performance and cost|Can use a combination of predictors of differing quality|

In this example, the 2BC simple predictor will be used for most branches, but if it is doing a poor job the branch is added to the Local predictor. Similarly it could also go to the Global predictor. Over time the CPU is "trained" on how to handle each branch.
![Hierarchical Predictor Example](https://i.imgur.com/o8jriH3.png)

## Return Address Stack (RAS)
For any branch, we need to know the direction (taken, not taken) and the target. The previous predictors have covered direction (BHT) and target (BTB) find for most types of branches (complex like `BEQ` and `BNE`, simple like `JUMP` and `CALL`, etc.). However, there is a type of branch, the function return, which is always taken (so direction prediction is fine), but the target is more difficult to predict, as it can be called from many different places. The BTB is not good at predicting the target when it could change each time.

The RAS is a predictor dedicated to predicting function returns. The idea is that upon each function call, we push the return address (PC+4) on the RAS. When we return, we pop from this stack to get the correct target address.

Why not simply use the stack itself? The prediction should be very close to the other predictors, and using a separate stack allows the predictor to be very small hardware structure and make the prediction very quickly.

What happens if we exceed the size of the RAS? Two choices:
- Don't push
- Wrap around - this is the best approach (main and top level functions do not consume the entire RAS)

In the end, remember this is another predictor and we are allowed to have mispredictions - the stack is still there and will work, just with misprediction cost. The goal is to optimize the greatest number of branches and returns, so the wrap-around approach is best.

### How do we know it's a `RET`?
This is all still during the IF phase, so how do we even know if the instruction is a `RET`? One simple way is to use a single-bit predictor to whether an instruction is a `RET` or not. This is very accurate (that PC is likely to always be a `RET`). 

Another approach is called "Predecoding". Most of the time instructions are coming from the processor cache and are only loaded from memory when not in the cache. This strategy involves storing part of the decoded instruction along with the instruction in cache. For example, if an instruction is 32-bits, maybe we store 33 bits, where the extra bit tells us if it is a `RET` or not. The alternative is decoding this every time the instruction is fetched, which becomes more expensive. Modern processors store a lot of information during the predecode step such that the pipeline can move quickly during execution.

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
