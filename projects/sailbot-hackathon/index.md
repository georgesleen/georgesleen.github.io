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

For a Sailbot-sponsored hackathon, our “elec-larpers” team picked a suitably
unreasonable goal: turn a YOLO model into a Sky130 ASIC in a weekend.

The path was hls4ml for model-to-C++, Bambu for C++-to-RTL, and LibreLane for
physical design. I brought up the Nix/LibreLane environment, got the generated
RTL into a reproducible handoff, and checked whether the result had any chance
of fitting.

It did not.

The generated top level contains 25 behavioral memories with two read and two
write ports, totaling `396.55 Mbits`. Even if I ignore the port problem and map
only the raw bit count onto the available single-port-ish Sky130 macro, the
lower bound is 24,204 SRAM macros and `6886.969 mm²` of macro area. That is
before paying for a real 2-read/2-write implementation, logic, routing, or
margin.

That killed the tapeout idea. The flow was still worth keeping: we got from the
model to generated RTL and into LibreLane, and I wrote a handoff that
reproduces both the setup and the failure, so nobody has to spend another
weekend finding the same memory wall.

## Glasses controller

I also worked on the control schematic for the team's smart-glasses idea. The
hierarchical sheets cover the controller, camera, IMU, haptics, and accelerator
interfaces. That is as far as it went: `glasses-control.kicad_pcb` contains an
empty KiCad board and no placed footprints or routing. It was schematic work,
not a fabricated or tested PCB.
