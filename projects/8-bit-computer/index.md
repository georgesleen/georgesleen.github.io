---
title: "8-Bit Computer"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "An 8-bit computer made of 7400-series logic, running programs in simulation."
thumbnail: "images/thumbnail.png"
media:
  - "images/general-register-render.png"
  - "images/program-memory-pcb.png"
  - "images/alu-schematic.png"
  - "images/full-computer.png"
  - "videos/simple-program.mp4"
date: 2024-08-01
status: "complete"
featured: true
tags: ["computer architecture", "simulation", "pcbs", "electrical", "mechanical"]
---

# 8-Bit Computer

Ben Eater's breadboard-computer series and Malvino's *Digital Computer Electronics* got me curious about what actually happens between an instruction hitting a register and a value showing up on the bus. I redrew a version of the machine in Logisim and started laying it out as modular 7400-series KiCad boards.

The architecture has an 8-bit shared bus, general and interface registers, a program counter, program memory, an instruction register, an ALU built from slices, and microcoded control logic. In simulation it's Turing-complete, and I wrote machine-code and custom-assembly programs to exercise it. The clip below runs one: load `4`, add `13`, output the result.

![Full simulated computer](images/full-computer.png)

<video src="videos/simple-program.mp4" controls style="width:100%; height:auto; display:block;"></video>

I didn't finish assembling the boards into a physical machine. The Logisim model runs the programs; the KiCad files were the next step.

_General-register PCB render:_

![General register render](images/general-register-render.png)

_Program-memory PCB:_

![Program memory PCB](images/program-memory-pcb.png)

_ALU schematic:_

![ALU schematic](images/alu-schematic.png)

[github.com/georgesleen/8-BitComputer](https://github.com/georgesleen/8-BitComputer)
