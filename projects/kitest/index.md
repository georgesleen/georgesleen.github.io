---
title: "KiTest"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "Simulation-based test harness for KiCad. DRC checks manufacturability, ERC checks wiring, KiTest checks intent."
thumbnail: ""
date: 2026-06-23
status: "active"
featured: false
tags: ["electrical", "pcbs", "simulation"]
media: []
---

# KiTest

A KiCad schematic can pass ERC and still be electrically wrong. ERC knows a net
is connected, but it doesn't know that a divider should produce 2.5 V or that an
amplifier should have a certain gain at 10 kHz. I kept rebuilding those SPICE
checks by hand, so I started KiTest to keep them beside the schematic.

**DRC checks manufacturability, ERC checks wiring, KiTest checks intent.**

## What works now

KiTest is a Rust workspace with a PyO3 package for writing tests under pytest.
It can export a schematic with `kicad-cli`, run ngspice in batch mode, and return
typed operating-point, transient, and AC results to Python.

The Python API now covers DC, pulse, sinusoidal, and AC sources; node lookup;
waveforms and spectra; absolute and percentage tolerances; settling, overshoot,
gain, and phase. There are pytest examples that run through KiCad export and
ngspice. The old page said the binding only exposed `version()`, which hasn't
been true for a while.

ngspice runs as a subprocess because its shared library keeps global,
single-instance state. Separate processes are much easier to run in parallel
without one test leaking into another. The Rust side still puts it behind a
`Backend` trait so I can replace it later.

## What isn't built yet

The plan has three levels: static netlist checks, cheap simulation smoke checks,
and full behavioral tests. Only parts of that exist today. Hierarchical-sheet
slicing, static connectivity assertions, broader smoke checks, MCU pin models,
Renode firmware co-simulation, Xyce, and a KiCad IPC frontend are all still
ideas.

I also plan to use one small RP2040 board as the dogfood design, starting with a
single analog test and adding the harder pieces one at a time. That board and
the later firmware-in-the-loop stages are plans, not finished KiTest features.

## Repository

[github.com/georgesleen/kitest](https://github.com/georgesleen/kitest)
