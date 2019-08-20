---
id: introduction
title: Introduction
sidebar_label: Introduction
---

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