---
title: "Phased Array Sonar"
layout: project.njk
description: "Ultrasonic phased-array transducer board with custom power tree, H-bridge driver, and RX amplifier chain."
thumbnail: ""
date: 2026-04-08
status: "active"
featured: false
tags: ["pcbs", "electrical", "power", "simulation"]
media: []
---

# Phased Array Sonar

## Overview

This project is a custom ultrasonic sonar system designed from scratch in
KiCad. The board transmits and receives ultrasonic pulses through a phased
array of piezoelectric transducers for applications like object detection and
distance measurement.

The project lives in a monorepo with the main board design, a development
board for prototyping TX/RX circuitry, a shared KiCad component library, and a
standalone SPICE simulation sub-project for the receive amplifier.

## System Architecture

The board has three major subsystems:

### Transmit (TX)

An H-bridge driver circuit generates 20 kHz drive signals for the transducer
array. The array uses Murata MA40S4S piezoelectric ultrasonic transducers
(40 kHz operating frequency, 10 mm diameter, 20 Vp-p max input). The top-level
schematic instantiates the 8-element transducer array three times, for a total
of 24 elements.

### Receive (RX)

SPV0142LR5H-1 MEMS microphones (Knowles, analog omnidirectional) feed into a
multi-stage amplifier chain. Each group of four MEMS mics connects to a quad
pre-amplifier, and each element has its own RX amplifier channel. The pre-amp
stage runs at a gain of 21 with a 120 MHz GBP op-amp.

A standalone SPICE simulation project (`rx_amp_sim/`) lets me iterate on the
RX amplifier design without dragging the full schematic into simulation.

### Power Tree

The power supply is hierarchical and supports two input sources through a power
mux with ideal diode ORing:

- USB-C (5 V / 900 mA)
- XT60 connector (battery or bench supply)

From the input, the power tree generates every rail the system needs:

- 5 V to 12 V boost (TPS55340PWPR)
- Inverting buck-boost for a negative rail (TPS63700DRCR)
- 3.3 V adjustable buck for digital logic
- Negative 10 V LDO (LM337IMPX)
- 12 V to 10 V LDO
- 2.75 V LDO

## Shared KiCad Library

The `sonar-library/` directory contains all shared symbols, footprints, 3D
models, and SPICE models. A Python setup script (`setup-kicad.py`, using PEP
723 inline metadata) registers the library with KiCad's global tables and
sets path variables in `kicad_common.json`. The script refuses to run while
KiCad is open to avoid config clobbering.

The library has over 40 symbols, 19 custom footprints, STEP models for the
connectors, and SPICE models for the TVS diode and boost converter.

## My Contributions

I am the primary contributor (24 of 41 commits). My work includes:

- **Power supply design.** I designed the full power tree: boost converters,
  inverting buck-boost (with some "questionable math" for the component
  calculations), LDOs, ideal diodes, and the input power mux.
- **TX H-bridge.** I built the 20 kHz H-bridge driver circuit from scratch.
- **Shared library and setup tooling.** I created the monorepo structure,
  wrote the shared KiCad library, and built the Python setup script.
- **Monorepo migration.** I merged the previously separate sonar-v1-pcb and
  tx-rx-dev-board repositories into a unified structure.

Joshua Himmens contributed the earliest schematic work, the RX receiver chain
(quad pre-amp, MEMS RX), and set up the RX amp SPICE simulation.

## Challenges

1. **Negative rail generation.** The RX amplifier chain needs a negative supply
   rail. I used a TPS63700 inverting buck-boost to generate it. The component
   calculations for the inductor and feedback resistors took multiple iterations
   to get right. I kept the old revision (`inverting_buck_boost_OLD.kicad_sch`)
   alongside the current one for reference.

2. **Library resolution across projects.** KiCad has two layers of library
   resolution: global (via `kicad_common.json` path variables) and
   project-local (via per-project `sym-lib-table`). Getting these to coexist
   without breaking paths across machines required the setup script and careful
   use of `${KIPRJMOD}` relative paths.

## Repository

[github.com/fizzy-sonar/sonar-hardware](https://github.com/fizzy-sonar/sonar-hardware)
