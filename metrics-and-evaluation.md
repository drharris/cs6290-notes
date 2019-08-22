---
id: metrics-and-evaluation
title: Metrics and Evaluation
sidebar_label: Metrics and Evaluation
---

[🔗Lecture on Udacity](https://classroom.udacity.com/courses/ud007/lessons/3650739106/concepts/last-viewed)

## Performance

* Latency (time start \\( \rightarrow \\) done)
* Throughput (#/second) (not necessarily 1/latency due to pipelining)

### Comparing Performance

"X is N times faster than Y"

Speedup = N = speed(X)/speed(Y)    
\\( \Rightarrow \\) N = Throughput(X)/Throughput(Y)    
\\( \Rightarrow \\) N = Latency(Y)/Latency(X)    
(notice Y and X for each)

### Speedup

Speedup > 1 \\( \Rightarrow \\) Improved Performance
* Shorter execution time
* Higher throughput

Speedup < 1 \\( \Rightarrow \\) Worse Performance

Performance ~ Throughput

Performance ~ 1/Latency

## Measuring Performance

How do we measure performance?

Actual User Workload:
* Many programs
* Not representative of other users
* How do we get workload data?

Instead, use Benchmarks

### Benchmarks

Programs and input data agreed upon for performance measurements. Typically there is a benchmark suite, comprised of multiple programs, each representative of some kind of application.

#### Benchmark Types
* Real Applications
  * Most Representative
  * Most difficult to set up
* Kernels
  * Find most time-consuming part of application
  * May not be easy to run depending on development phase
  * Best to run once a prototype machine is available
* Synthetic Benchmarks
  * Behave similarly to kernels but simpler to compile
  * Typically good for design studies to choose from design
* Peak Performance
  * In theory, how many IPS
  * Typically only good for marketing

#### Benchmark Standards

A benchmarking organization takes input from academia, user groups, and manufacturers, and curates a standard benchmark suite. Examples: TPC (databases, web), EEMBC (embedded), SPEC (engineering workstations, raw processors). For example, SPEC includes GCC, Perl, BZIP, and more.

### Summarizing Performance

#### Average Execution Time

|   | Comp X | Comp Y | Speedup |
|---|--------|--------|---------|
| App A |  9s | 18s | 2.0 |
| App B | 10s |  7s | 0.7 |
| App C |  5s | 11s | 2.2 |
| AVG   |  8s | 12s | 1.5 |

If you simply average speedups, you get 1.63 instead. Speedup of average execution times is not the same as simply averaging speedups on individual applications.

#### Geometric Mean

If we want to be able to average speedups, we need to use geometric means for both average times and speedups. This results in the same value whether you are taking the geometric mean of individual speedups, or speedup of geometric mean of execution times.

For example, in the table above we would obtain:
|   | Comp X | Comp Y | Speedup |
|---|--------|--------|---------|
| Geo Mean |  7.66s | 11.15s | 1.456 |

Geometric mean of speedup values (2.0, 0.7, 2.2) also result in 1.456.

$$ \text{geometric mean} = \sqrt[n]{a_1\*a_2\*...a_n} $$

As a general rule, if you are trying to average things that are ratios (speedups are ratios), you cannot simply average them. Use the geometric mean instead.

## Iron Law of Performance

**CPU Time** = (# instructions in the program) * (cycles per instruction) * (clock cycle time)

Each component allows us to think about the computer architecture and how it can be changed:
* Instructions in the Program
  * Algorithm
  * Compiler
  * Instruction Set
* Cycles Per Instruction
  * Instruction Set
  * Processor Design
* Clock Cycle Time
  * Processor Design
  * Circuit Design
  * Transistor Physics

### Iron Law for Unequal Instruction Times

When instructions have different number of cycles, sum them individually:
$$ \sum_i (IC_i\* CPI_i) * \frac{\text{time}}{\text{cycle}} $$

where \\( IC_i \\) is the instruction count for instruction \\( i \\), and \\(CPI_i\\) is the cycles for instruction \\( i \\).

## Amdahl's Law

Used when only part of the program or certain instructions. What is the overall affect on speedup?

$$ speedup = \frac{1}{(1-frac_{enh}) + \frac{frac_{enh}}{speedup_{enh}}} $$

where \\( frac_{enh} \\) represents the fraction of the execution **TIME** enhanced by the changes, and \\( speedup_{enh} \\) represents the amount that change was sped up.

NOTE: Always ensure the fraction represents TIME, not any other quantity (cycles, etc.). First, convert changes into execution time before the change, and execution time after the change.

### Implications

Compare these enhancements:
* Enhancement 1
  * Speedup of 20x on 10% of time
  * \\( \Rightarrow \\) speedup = 1.105
* Enhancement 2
  * Speedup of 1.6x on 80% of time
    * \\( \Rightarrow \\) speedup = 1.43

Even an infinite speedup in enhancement 1 only yields 1.111 overall speedup. 

Takeaway: Make the common case fast

### Lhadma's Law

* Amdahl: Make common case fast
* Lhadma: Do not mess up the uncommon case too badly

Example:
* Improvement of 2x on 90%
* Slow down rest by 10x
* Speedup = \\( \frac{1}{\frac{0.1}{0.1} + \frac{0.9}{2}} = \frac{1}{1+0.45} = 0.7 \\)
* \\( \Rightarrow \\) Net slowdown, not speedup.

### Diminishing Returns

Consequence of Amdahl's law. If you keep trying to improve the same area, you get diminishing returns on the effort. Always reconsider what is now the dominant part of the execution time. 

![Diminishing Returns](https://i.imgur.com/SzjXnRS.png)