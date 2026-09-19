---
title: "Sailbot Hackathon"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "A YOLO-to-RTL experiment, a reproducible Sky130 flow, and an unfinished smart-glasses schematic."
thumbnail: ""
date: 2026-03-15
status: "complete"
featured: false
tags: ["electrical", "pcbs", "embedded"]
media: []
---

# Sailbot Hackathon

March 2026, Sailbot-sponsored hackathon, "elec-larpers" team. Our goal was to put a YOLO object-detection model onto a custom Sky130 ASIC.

The pipeline was YOLO → hls4ml → C++ → Bambu → Verilog RTL → LibreLane → Sky130. I brought up the Nix-based LibreLane environment, got the generated RTL through the handoff, and ran a feasibility check on whether the design could plausibly fit.

It couldn't. The generated top level instantiates 25 behavioral 2-read/2-write SRAM macros totalling `396.55 Mbits`. Even if I ignore the port mismatch and map the raw bit count onto Sky130's available single-port-ish macros, the lower bound is 24,204 SRAM macros and about `6886.969 mm²` of macro area — before anything for the actual 2R2W behavior, logic, routing, or margin. The useful output wasn't a tapeout candidate; it was the reproducible LibreLane flow and an explicit number for why the design didn't fit.

## Glasses controller

Alongside the ASIC work, I did some schematic work on a "glasses-control" PCB for the team's smart-glasses idea. The hierarchical sheets cover the STM32 controller, camera interfaces, IMU, haptics, and the accelerator interface. The board file (`glasses-control.kicad_pcb`) is empty — no footprints placed, no routing, no fabrication. It stopped at schematic.
