---
title: "8-Bit Computer"
layout: project.njk
description: "A custom 8-bit computer designed from discrete components."
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

## Overview
This project was a ground-up implementation of a fully custom **8-bit computer**, built from discrete logic components and custom PCB modules.  
The system is **Turing-complete**, capable of executing a minimal instruction set, and was developed as a way to explore computer architecture at the gate and register-transfer level.

The design takes inspiration from [Ben Eater’s breadboard computer](https://eater.net/), but expands it with a modular PCB-based implementation, Logisim simulations, and a structured hardware layout.

---

## Contributions
- **Architecture Definition**: Designed a simple but complete 8-bit instruction set and system architecture (registers, ALU, program counter, memory, control logic).
- **Schematic & PCB Design**: Created KiCad schematics and PCB layouts for the computer’s core subsystems (general registers, ALU, program memory).
- **Simulation & Verification**: Validated the instruction set and control logic in Logisim before committing to hardware.
- **System Integration**: Built and tested each module in isolation, then integrated them onto a shared bus.
- **Demonstration**: Successfully ran small programs demonstrating arithmetic operations and data output.

---

## Technical Highlights
- **Registers**: Designed custom register boards with tri-state buffer outputs and synchronous load.
- **ALU**: Implemented addition, subtraction, and bitwise logic operations with discrete logic gates.
- **Memory**: Designed a program ROM PCB and accompanying RAM for instruction storage.
- **Control Logic**: Implemented fetch–decode–execute sequencing via microcoded control signals.
- **Bus Design**: Shared system bus with tri-state management across modules.

---

## Media

### Repository
[GitHub – 8-Bit Computer](https://github.com/georgesleen/8-BitComputer)

### Hardware & Schematics
- *General Register*  
  ![General register render](images/general-register-render.png)

- *Program Memory PCB*  
  ![Program memory pcb](images/program-memory-pcb.png)

- *ALU Schematic*  
  ![ALU schematic](images/alu-schematic.png)

### Simulation
- *Full simulated computer*  
  ![Full Simulated Computer](images/full-computer.png)

- *Simple program execution:*

    1. Load `4` into Register A
    2. Add `13` to Register A
    3. Output result to output register

  <video src="videos/simple-program.mp4" controls style="width:100%; height:auto; display:block;"></video>

---

## Reflection
This project provided a first-principles understanding of how digital computers operate at the logic level.  
Key takeaways included:
- How simple bit manipulation and storage gives rise to the complex programs run by computers.
- Using simulation to de-risk hardware design.
- Appreciating the tradeoffs between simplicity, expandability, and physical wiring complexity.

If extended further, this design could support conditional branching, subroutines, or pipelining.
