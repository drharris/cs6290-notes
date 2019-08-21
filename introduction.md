---
id: introduction
title: Introduction
sidebar_label: Introduction
---

[Lecture on Udacity](https://classroom.udacity.com/courses/ud007/lessons/3627649022/concepts/last-viewed)

## Computer Architecture

### What is it?

Architecture - design a *building* that is well-suited for its purpose    
Computer Architecture - design a *computer* that is well-suited for its purpose.

### Why do we need it?

1. Improve Performance (e.g. speed, battery life, size, weight, energy effficiency, ...)
2. Improve Abilities (3D graphics, debugging support, security, ...)

Takeaway: Computer Architecture takes available hardware (fabrication, circuit designs) to create faster, lighter, cheaper, etc. computers.

## Technology Trends

If you design given current technology/parts, you get an obsolete computer by the time the design is complete. Must take into account technological trends and anticipate future technology

### Moore's Law

Used to predict technology trends. Every 18-24 months, you get 2x transistors for same chip area:    
\\( \Rightarrow \\) Processor speed doubles    
\\( \Rightarrow \\) Energy/Operation cut in half    
\\( \Rightarrow \\) Memory capacity doubles

### Memory Wall

Consequence of Moore's Law. IPS (instructions/sec) and Capacity double every 2 years. If Latency only improves 1.1x every two years, there is a gap between latency and speed/capacity, called the Memory Wall. Caches are used to fill in this gap.

![Memory Wall Graph](https://i.imgur.com/RMSndOW.png)

## Processor Speed, Cost, Power

|    | Speed | Cost | Power |
| ---| ----- | ---- | ----  |
|    |   2x  |  1x  |  1x   |
| OR | 1.1x  | .5x  | .5x   |

Improvements may differ, it's not always speed. Which one of the above do you really want? It depends.

## Power Consumption

Two kinds of power a processor consumes.
1. Dynamic Power - consumed by activity in a circuit
2. Static Power - consumed when powered on, but idle

### Active Power
Can be computed by

$$P = \tfrac 12 \*C\*V^2\*f\*\alpha$$
Where
$$\\\\C = \text{capacitance}\\\\P = \text{power supply voltage}\\\\f = \text{frequency} \\\\ \alpha = \text{activity factor (some percentage of transistors active each clock cycle)}$$

For example, if we cut size/capacitance in half but put two cores on a chip, the active power is relatively unchanged. To get power improvements, you really need to lower the voltage.

### Static Power

The power it takes to maintain the circuits when not in use.
![Static Power](https://i.imgur.com/Db7NwSj.png)
Decreasing the voltage means the reduced electrical "pressure" on a transistor results in more static power usage. However, increasing the voltage means you consume more active power. There is a minimum at which total power (static + dynamic) occurs.

## Fabrication Cost and Yield

Circuits are printed onto a wafer in silicon, divided up into small chips, and packaged. The packaged chip is tested and either discarded or shipped/sold. Larger chips could have more fallout due to probabilities of using larger areas of the wafer, and thus cost more.

$$ \text{fabrication yield} =  \tfrac{\text{working chips}}{\text{chips on wafer}} $$

![Fabrication Yield](https://i.imgur.com/vIIzt0I.png)

Fabrication cost example: If a wafer costs $5000, and has approximately 10 defects, how much does each chip cost in the following example? Remember \\( \text{chip cost} = \tfrac{\text{wafer cost}}{\text{fabrication yield} } \\)

| Size | Chips/Wafer | Yield |  Cost  | 
|------|-------------|-------|--------|
|Small |    400      |  390  | $12.80 |
|Large |     96      |   86  | $58.20 |
| Huge |     20      |   11  | $454.55|

Benefit from Moore's Law in two ways:
* Smaller \\( \Rightarrow \\) much lower cost
* Same Area \\( \Rightarrow \\) faster for same cost