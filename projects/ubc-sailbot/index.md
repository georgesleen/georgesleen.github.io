---
title: "UBC Sailbot"
layout: project.njk
description: "Communication firmware, hackathon ASIC exploration, and embedded systems work for UBC Sailbot's autonomous sailboat Polaris."
thumbnail: "media/polaris-imu-pcb-orthographic.png"
date: 2026-08-16
status: "active"
featured: false
tags: ["embedded", "firmware", "electrical"]
media:
  - media/polaris-imu-pcb-orthographic.png
  - media/gdb-debugging.png
  - media/nmea-messages-printing.png
---

# UBC Sailbot

## Overview

UBC Sailbot builds an autonomous sailboat called Polaris. I joined the team in
September 2025 and work on the embedded systems that keep the boat running: the
shared communication firmware that ties the subsystems together, and the
[PLRS-IMU](/projects/plrs-imu/) heading sensor fusion system (covered on its
own page).

---

## Communication Module Firmware

The boat's subsystems (rudder controller, wingsail controller, sensor module,
power distribution) each run on STM32U575 Nucleo boards. The communication
module firmware is the shared codebase for all of them.

My contributions include:

- **NMEA0183 parsing** for the LCJ CV7 wind sensor. I wrote clean parsers with
  typed structs so the data self-documents. No more raw fixed-point integers
  with comments explaining how to decode them.
- **Rudder link protocol.** I designed a COBS-framed serial protocol that feeds
  the fused heading from the IMU to the rudder controller.
- **Rudder equivalence analysis.** When a teammate refactored the rudder control
  model, I did a forensic comparison and found two real bugs: a doubled integral
  increment and uninitialized state.
- **Integration and branch management.** I merged the wingsail, rudder, PDB,
  and sensor module branches into one coherent state, documented all branch
  tips, and wrote the git flow and testing workflow for the team.
- **CI pipeline.** I set up GitHub Actions for `.ioc` peripheral checking and
  host unit tests.

I started on this codebase in October 2025, learning the team's STM32CubeIDE
workflow and the Nucleo hardware. My early work sessions involved getting UART
printing to work, debugging I2C and NMEA protocol issues with an Analog
Discovery 3, and writing unit tests for the wind sensor parser.

---

## Sailbot Hackathon

In March 2026, I participated in a sailbot hackathon with the "elec-larpers"
team. Our project explored putting a YOLO object detection model onto a custom
ASIC. The pipeline went: YOLO model (Python) to C++ via hls4ml, to Verilog RTL
via Bambu, and finally to a Sky130 ASIC via LibreLane.

I set up the Nix-based LibreLane environment for the ASIC flow and did the
feasibility analysis. The result: the design requires approximately 396 Mbits
of SRAM (about 24,000 SRAM macros, 6,887 mm^2 of macro area). Not tapeout
ready, but a useful exercise in understanding the full ML-to-silicon pipeline.

I also designed a "glasses-control" PCB with an STM32, camera interfaces,
haptic feedback, and an IMU. It was a one-day build during the hackathon.

---

## Repositories

- PLRS-IMU (separate page): [PLRS-IMU project](/projects/plrs-imu/)
- Communication firmware: [github.com/UBCSailbot/com-module-firmware](https://github.com/UBCSailbot/com-module-firmware)
