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

A KiCad schematic can pass ERC and still be electrically wrong. ERC will tell me the net exists; it can't tell me a divider should sit at 2.5 V, an amplifier should have a specific gain at 10 kHz, or a transient should settle within a tolerance. I kept rebuilding those SPICE setups by hand every time the schematic changed, so I started KiTest to keep them beside the design.

The framing I like: **DRC checks manufacturability, ERC checks wiring, KiTest checks intent.**

## What works

KiTest is a Rust workspace with a PyO3 package for pytest-facing testbenches. Today it can export a KiCad schematic through `kicad-cli`, drive ngspice in batch mode, and hand typed operating-point, transient, and AC results back to Python. The Python binding is functional now, not just `version()`; it exposes DC/pulse/sinusoidal/AC sources, node lookups, waveforms and spectra, and tolerance-aware settling, overshoot, gain, and phase checks. There are pytest examples that run the whole thing end-to-end through KiCad export and ngspice.

ngspice runs as a subprocess rather than through libngspice, because libngspice has single-instance global state that makes parallel tests painful. The subprocess also keeps KiTest license-independent from ngspice's GPL. A `Backend` trait leaves room for an FFI backend or Xyce later.

## What isn't built

The long-term design has three tiers: static netlist checks, cheap simulation smoke checks, and full behavioral testbenches. Only pieces of that exist. Hierarchical-sheet slicing, most of the static connectivity assertions, more of the smoke checks, behavioral MCU pin models, Renode-based firmware-in-the-loop, an Xyce backend, and a KiCad IPC plugin frontend are all still ahead.

There's a planned RP2040 dogfood board that's supposed to drive the work one small feature at a time — LED for the walking skeleton, PWM/RC "DAC" and an ADC-input node for the analog tiers, firmware-in-the-loop after that. It exists on paper, not in the KiTest tree.

[github.com/georgesleen/kitest](https://github.com/georgesleen/kitest)
