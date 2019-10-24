---
id: storage
title: Storage
sidebar_label: Storage
---

[🔗Lecture on Udacity (30 min)](https://classroom.udacity.com/courses/ud007/lessons/872590121/concepts/last-viewed)

## Storage
Reasons we need it:
* Files - Programs, data, settings
* Virtual Memory

We care about:
* Performance
  * Throughput - improving (not as quickly as processor speed)
  * Latency - improving (very slowly, more so than DRAM)
* Reliability

Types we can use are very diverse
* Magnetic Disks
* Optical Disks
* Tape
* Flash
* ... etc.

## Magnetic Disks
Multiple platters rotate around a spindle. A head assembly sits beside these platters, and has a head that is used to read both sides of the magnetic platter surface. Where the head is positioned on the platter is called a track - this is the circle of data the head can access while in that position as the platters rotate. A cylinder is the collation of the same track across multiple platters. The head will move position which allows access to other tracks.

![magnetic disks](https://i.imgur.com/CB5bwaK.png)

The data will be organized into position order along a track, and divided into sectors. Each sector contains a preamble, some data, and some kind of checksum or error correction code.

Disk capacity then can be represented by the multiplication of:
- number of platters x 2 (surfaces)
- number of tracks/surface (cylinders)
- number of sectors/track
- number of bytes/sector

### Access time for Magnetic Disks
(If the disk is spinning already)
- Seek Time - move the head assembly to the correct cylinder
- Rotational Latency - Wait for the start of our sector to get under the head
- Data Read - read until end of sector seen by head
- Controller Time - checksum, determine sector is ok
- I/O Bus Time - get the data into main memory

...plus a significant Queuing Delay (wait for previous accesses to complete before we can move the head again)

### Trends for Magnetic Disks
- Capacity
  - 2x per 1-2 years
- Seek Time
  - 5-10ms, very slow improvement (primarily due to shrinking disk diameter)
- Rotation: 
  - 5,000 RPM \\(\rightarrow\\) 10,000 RPM \\(\rightarrow\\) 15,000 RPM
  - Materials and noise improvements
  - Improves slowly
- Controller, Bus
  - Improves at OK rate
  - Currently a small fraction of overall time

Overall performance is dominated by materials and mechanics, not something like Moore's Law - so it is very difficult to rapidly improve on this technology.

## Optical Disks
Similar to hard disk platter, except we use a laser instead of a magnetic head. Additionally, where a hard disk needs enclosing to keep contaminants out, this is not as important on an optical disk. So being open (insert/remove any disk) allows portability, as does standardization of technologies.

However, improving this technology is not very easy because portability requires standards that must be agreed upon, which takes time. Then products still have to be made to address the new technologies, and will take even more time to be adopted by consumers.

## Magnetic Tape
* Backup (secondary storage)
* Large capacity, replaceable
* Sequential access (good for large sequential data, poor for virtual memory)
* Currently dying out
  * Low production volume \\(\Rightarrow\\) cost not dropping as rapidly as disks
  * Cheaper to use more disks rather than invest in new equipment and the tapes to use them (and USB hard drives)

## Using RAM for storage
* Disk about 100x cheaper per GB
* DRAM has about 100,000x better latency

Solid-State Disk (SSD)
* DRAM + Battery
  * Fast!
  * Reliable
  * Extremely Expensive
  * Not good for archiving (must be powered)
* Flash
  * Low Power
  * Fast (but slower than DRAM)
  * Smaller Capacity (GB vs TB)
  * Keeps data alive without power

## Hybrid Magnetic Flash
Combine magnetic disk with Flash
* Magnetic Disk
  * Low $/GB
  * Huge Capacity
  * Power Hungry
  * Slow (mechanical movement)
  * Sensitive to impacts while spinning
* Flash
  * Fast
  * Power Efficient
  * No moving parts

Use both!
* Use Flash as cache for disk
* Can potentially power down the disk when what we need is in cache

## Connecting I/O Devices
![connecting I/O devices](https://i.imgur.com/Jt1d7Xm.png)
Main point: Things closer to the CPU may need full speed of the bus. Things farther away or naturally slower (e.g. storage, USB) will not utilize as much of the speed, so are more interested in standardization of the bus/controller technology, which grows at slower rates.