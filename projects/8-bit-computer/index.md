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

I built this after watching Ben Eater's breadboard-computer series and reading
Malvino's *Digital Computer Electronics*. I wanted to know what actually
happened between an instruction entering the register and a value appearing on
the bus, so I drew the whole computer in Logisim and then split it into
7400-series KiCad modules.

The machine has an 8-bit shared bus, general and interface registers, a program
counter, program memory, an instruction register, an ALU, and microcoded control
logic. The ALU is built out of slices for the arithmetic and logic operations,
and the control logic sequences all the bus transfers behind each fetch, decode,
and execute step.

The simulated computer is Turing-complete and runs programs I wrote in machine
code and a small custom assembly language. The clip below loads `4` into a
register, adds `13`, and sends the answer to the output register.

![Full simulated computer](images/full-computer.png)

<video src="videos/simple-program.mp4" controls style="width:100%; height:auto; display:block;"></video>

## From simulation to boards

Working it out in simulation first meant I could settle the instruction path and
control timing before drawing any boards. The repo has KiCad schematics and PCB
layouts for the ALU, registers, program counter, program memory, instruction
register, and control logic.

_General-register PCB render:_

![General register render](images/general-register-render.png)

_Program-memory PCB:_

![Program memory PCB](images/program-memory-pcb.png)

_ALU schematic:_

![ALU schematic](images/alu-schematic.png)

I never finished assembling all of these boards into one computer. The programs
ran in Logisim; the KiCad files show how I planned to turn each simulated block
into hardware.

## Repository

[github.com/georgesleen/8-BitComputer](https://github.com/georgesleen/8-BitComputer)
