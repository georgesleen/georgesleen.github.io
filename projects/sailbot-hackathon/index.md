---
title: "Sailbot Hackathon"
layout: project.njk
description: "YOLO-to-ASIC pipeline and glasses-control PCB from a sailbot-sponsored hackathon."
thumbnail: ""
date: 2026-03-15
status: "complete"
featured: false
tags: ["electrical", "pcbs", "embedded"]
media: []
---

# Sailbot Hackathon

In March 2026 I did a hackathon sponsored by UBC Sailbot with the
"elec-larpers" team. We tried to put a YOLO object detection model onto a
custom ASIC.

## YOLO to ASIC

The pipeline went: YOLO model (Python) to C++ via hls4ml, to Verilog RTL via
Bambu, to a Sky130 ASIC via LibreLane. I set up the Nix-based LibreLane
environment and did the feasibility analysis. The result: the design needs about
396 Mbits of SRAM (roughly 24,000 SRAM macros, 6,887 mm^2 of macro area). Not
tapeout ready, but a useful exercise in understanding the full ML-to-silicon
pipeline.

## Glasses-control PCB

I also designed a "glasses-control" PCB with an STM32, camera interfaces,
haptic feedback, and an IMU. It was a one-day build during the hackathon.
