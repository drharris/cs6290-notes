---
id: fault-tolerance
title: Fault Tolerance
sidebar_label: Fault Tolerance
---

[🔗Lecture on Udacity (1.5hr)](https://classroom.udacity.com/courses/ud007/lessons/872590122/concepts/last-viewed)

## Dependability
Quality of delivered service that justifies relying on the system to provide that service.
- Specified Service = what behavior should be
- Delivered Service = actual behavior

System has components (modules)
- Each module has an ideal specified behavior

## Faults, Errors, and Failures (+ Example)
* **Fault** - module deviates from specified behavior
  * Example: Programming mistake
    * Add function that works fine, except 5+3 = 7
    * Latent Error (only a matter of time until activated)
* **Error** - actual behavior within system differs from specified behavior
  * Example: (Activated Fault, Effective Error)
    * We call add() with 5 and 3, get 7, and put it in some variable
* **Failure** - System Deviates from specified behavior
  * Example: 5+3=7 -> Schedule a meeting for 7am instead of 8am

An error starts with a fault, but a fault may not necessarily become an error. A failure starts with an error, but an error may not necessarily result in a failure. Example: `if(add(5, 3))>0)` causes an error since the fault is activated, but not a failure since the end result is not affected.

## Reliability, Availability
System is in one of two states:
* Service Accomplishment
* Service Interruption

Reliability:
* Measure continuous Service Accomplishment
* Mean Time to Failure (MTTF)

Availability:
* Service Accomplishment as a fraction of overall time
* Need to know: Mean Time to Repair (MTTR)
* Availability = \\(\frac{MTTF}{MTTF+MTTR}\\)

## Kinds of Faults
By Cause:
* HW Faults - HW fails to perform as designed
* Design Faults - SW bugs, HW design mistakes (FDIV bug)
* Operation Faults - Operator, user mistakes
* Environmental Faults - Fire, power failures, sabotage, etc.

By Duration:
* Permanent - Once we have it, it doesn't get corrected (wanted to see what's inside processor, and now it's in 4 pieces)
* Intermittent - Last for a while, but recurring (overclock - temporary crashes)
* Transient - Something causes things to stop working correctly, but it fixes itself eventually

## Improving Reliability and Availability
* Fault Avoidance
  * Prevent Faults from occurring at all
  * Example: No coffee in server room
* Fault Tolerance
  * Prevent Faults from becoming Failures
  * Example: Redundancy, e.g. ECC for memory
* Speed up Repair Process (availability only)
  * Example: Keep a spare hard disk in drawer

## Fault Tolerance Techniques
* Checkpointing (Recover from error)
  * Save state periodically
  * If we detect errors, restore the saved state
  * Works well for many transient and intermittent failures
  * If this takes too long, it has to be treated like a service interruption
* 2-Way Redundancy (Detect error)
  * Two modules do the same work and compare results
  * Roll back if results are different
* 3-Way Redundancy (Detect and recover from error)
  * 3 modules (or more) do the same work and vote on correctness
  * Fault in one module does not become an error overall.
  * Expensive - 3x the hardware required, but can tolerate *any* fault in one module

## N-Module Redundancy
* N=2 - Dual-Module Redundancy
  * Detect but not correct one faulty module
* N=3 - Triple-Module Redundancy
  * Correct one faulty module
* N=5 - (example: space shuttle)
  * 5 computers perform operation and vote
  * 1 Wrong Result \\(\Rightarrow\\) normal operation
  * 2 Wrong Results \\(\Rightarrow\\) abort mission
    * Still no failure from this: 3 outvote the 2
  * 3 Wrong Results: failure can be catastrophic (too many broken modules)
    * Abort with 2 failures so that this state should never be reached

## Fault Tolerance for Memory and Storage
* Dual/Triple Module Redundancy - Overkill (typically better for computation)
* Error Detection, Correction Codes
  * Parity: One extra bit (XOR of all data bits)
    * Fault flips one bit \\(\Rightarrow\\) Parity does not match data
  * ECC: example - SECDED (Single Error Correction, Double Error Detection)
    * Can detect and fix any single bit flip, or can only detect any dual bit flip
    * Example: ECC DRAM modules
  * Disks can use even fancier codes (e.g. Reed-Solomon)
    * Detect and correct multiple-bit errors (especially streaks of flipped bits)
* RAID (for hard disks)

## RAID - Redundant Array of Independent Disks
* Several disks playing the role of one disk (can be larger and/or more reliable than the single disk)
* Each disk detects errors using codes
  * We know which disk has the error
* RAID should have:
  * Better performance
  * Normal Read/Write accomplishment even when:
    * It has a bad sector
    * Entire disk fails
* RAID 0, 1, etc...

### RAID 0: Striping (to improve performance)
Disks can only read one track at a time, since the head can be in only one position. RAID 0 takes two disks and "stripes" the data across each disk such that consecutive tracks can be accessed simultaneously with the head in a single position. This results in up to 2x the data throughput and reduced queuing delay.

However, reliability is worse than a single disk:
* \\(f\\) = failure rate for a single disk
  * failures/disk/second
* Single-Disk MTTF = \\(\frac{1}{f}\\)  (MTTDL: Mean time to data loss = \\(MTTF_1\\))
* N disks in RAID0
  * \\(f_N = N*f_1 \Rightarrow MTTF_N = MTTDL_N = \frac{MTTF_1}{N}\\)
  * 2 Disks \\( \Rightarrow  MTTF_2 = \frac{MTTF_1}{2} \\)

### RAID 1: Mirroring (to improve reliability)
Same data on both disks
* Write: Write to each disk
  * Same performance as 1 disk alone
* Read: Read any one disk
  * 2x throughput of one disk alone
* Can tolerate any faults that affect one disk
  * Two copies -> Detect Error ???
    * Not true in this case, because ECC on each sector lets us know which one has a fault

Reliability: 
* \\(f\\) = failure rate for a single disk
  * failures/disk/second
* Single-Disk MTTF = \\(\frac{1}{f}\\)  (MTTDL: Mean time to data loss = \\(MTTF_1\\))
* 2 disks in RAID1
  * \\(f_N = N*f_1 \Rightarrow\\) 
    * both disks OK until \\(\frac{MTTF_1}{2}\\)
    * remaining disk lives on for \\(MTTF_1\\) time
  * \\(MTTDL_{RAID1-2} = \frac{MTTFF_1}{2}+MTTF_1\\) (Assumes no disk replaced)
* But we do replace failed disks!
  * Both disks ok until \\(\frac{MTTF_1}{2}\\)
  * Disk fails, have one OK disk for \\(MTTR_1\\)
  * Both disks ok again until \\(\frac{MTTF_1}{2}\\)
  * So, overall MTTDL :
    * (when \\(MTTR_1 \ll MTTF_1\\), probability of second disk failing during MTTR = \\(\frac{MTTR_1}{MTTF_1}\\))
    * \\(MTTDL_{RAID1-2} = \frac{MTTF_1}{2} * \frac{MTTF_1}{MTTR_1}\\) (second factor is 1/probability)

### RAID 4: Block-Interleaved Parity
* N disks
  * N-1 contain data, striped like RAID 0
  * 1 disk has parity blocks

| Disk 0   |   | Disk 1    |   | Disk 2    |   | Disk 3      |
|----------|---|-----------|---|-----------|---|-------------|
| Stripe 0 | ⊕ | Stripe 1 | ⊕ | Stripe 2 | = | Parity 0,1,2 |
| Stripe 3 | ⊕ | Stripe 4 | ⊕ | Stripe 5 | = | Parity 3,4,5 |

Data from each stripe is XOR'd together to result in parity information on disk 3. If any one of the disks fail, the data can then be reconstructed by XOR-ing all the other disks, including parity.

* Write: Write 1 data disk and parity disk read/write
* Read: Read 1 disk

Performance and Reliability:
* Reads: throughput of N-1 disks
* Writes: 1/2 throughput of single disk (primary reason for RAID 5)
* MTTF:
  * All disks ok for \\(\frac{MTTF_1}{N}\\)
    * If no repair, we are left with N-1 disk array: + \\(\frac{MTTF_1}{N-1} \rightarrow\\) Bad idea
    * Repair \\(\Rightarrow\\) chance of another failing during repair: \\(\frac{MTTF_1}{N-1} \\): Multiply by this factor over \\(MTTR_1\\)
  * \\(MTTF_{RAID4} = \frac{MTTF_1 \* MTTF_1}{N\*(N-1)\*MTTR_1}\\)

Reads: [🎥 View Lecture Video (2:19)](https://www.youtube.com/watch?v=3QXaSzM2fE8)
- If we compute the XOR of the old vs new data we are writing, we get the bit flips we're making to the data. If we then XOR this against the parity block, we perform those same bit flips on the parity data and get the new parity information.
- Thus, the parity disk is a bottleneck for writes (multiple disks may be updating it) \\(\Rightarrow\\) RAID5

### RAID 5: Distributed Block-Interleaved Parity
* Like RAID 4, but parity is spread among all disks:
    | Disk 0   |   | Disk 1    |   | Disk 2    |   | Disk 3      |
    |----------|---|-----------|---|-----------|---|-------------|
    | Stripe 0 | ⊕ | Stripe 1 | ⊕ | Stripe 2 | = | Parity 0,1,2 |
    | Parity 3,4,5 | = | Stripe 3 | ⊕ | Stripe 4 | ⊕ | Stripe 5 |
    | Stripe 6 |  | Parity 6,7,8 |  | Stripe 7 |  | Stripe 8 |
    | Stripe 9 |  | Stripe 10 |  | Parity 9,10,11 |  | Stripe 11|
    | Stripe 12 | ⊕ | Stripe 13 | ⊕ | Stripe 14 | = | Parity 12,13,14 |
* Read Performance: N * throughput of 1 disk
* Write Performance: 4 accesses/write, but distributed: N/4 * throughput of 1
* Reliability: same as RAID4 - if we lose any one disk, we have a problem
* Capacity: same as RAID4 (still sacrifice one disk worth of parity over the array)

Key takeaway: We should always choose RAID5 over RAID4, since we lose nothing with reliability or capacity, and gain in throughput, all without additional hardware requirements.

### RAID 6?
* Two "parity" blocks per group
  * Can work when 2 failed stripes/group
  * One parity block
  * Second is a different type of check-block
  * When 1 disk fails, use parity
  * When 2 disks fail, solve equations to retrieve data
* RAID 5 vs. RAID 6
  * 2x overhead
  * More write overhead (6/WR vs 4/WR)
  * Only helps reliability when disk fails, then another fails before we replace the first one (low probability, thousands of years MTTDL)

RAID6 is an overkill?
* RAID5: disk fails, 3 days to replace
  * Very low probability of another failing in those 3 days (assuming independent failure)
* Failures can be related!
  * RAID5, 5 disks, 1 disk fails (#2)
  * System says "Replace disk #2"
  * Operator gets replacement disk and inserts it into spot 2
  * But... numbering was 0,1,2,3,4! Operator pulled the wrong disk.
  * Now we have two failed disks (one hard failure, one operator error)
  * RAID6 prevents both a single or double disk failure.

The above scenario sounds silly, but with very long MTTF values, replacing a RAID disk becomes a rare activity, and is prone to operator error. A real-life personal scenario I (author of these notes) encountered was not properly grounding myself when replacing a disk in a hot-swappable RAID5. When inserting it into the housing, my theory is that I discharged static electricity into the disk below the one I was replacing, and the array failed to rebuild due to an error on that disk. Thankfully we were able to move the platters from that drive into a new one and the array rebuilt ok, avoiding the need to use week-old backups. So, RAID6 is indeed a bit overkill, but could have prevented this scenario, which resulted in lots of system downtime.


*[MTTDL]: Mean Time to Data Loss
*[MTTF]: Mean Time to Failure
*[MTTR]: Mean Time to Repair