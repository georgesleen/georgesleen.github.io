---
title: "KiTest"
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

The idea behind KiTest is simple: a schematic captures the design, but nowhere
does it capture the intended behavior. What voltage should appear at this node?
How fast should the output settle? What is the gain at 10 kHz? Today, engineers
verify these properties by manually building SPICE test setups every time they
change the circuit. KiTest makes those tests persistent, one-command re-runnable
artifacts that live next to the design files.

The framing I like: **DRC checks manufacturability, ERC checks wiring, KiTest
checks intent.**

## How it works

The pipeline has four stages:

1. Export a KiCad schematic to a SPICE netlist via `kicad-cli`
2. Inject test-side stimulus (DC supplies, AC sources) into the netlist
3. Run an analysis (operating point, transient, or AC sweep) through ngspice
4. Parse the results and check them against assertions

A test for a resistor divider: export the schematic, inject a 5 V supply, run
an operating point analysis, assert that the output settles to 2.5 V within 1%.

## Architecture

The core is Rust (edition 2024), structured as a Cargo workspace with two
crates:

- **kitest** has the engine: a `Backend` trait with three methods (`run_op`,
  `run_tran`, `run_ac`), each returning a typed result. The ngspice backend
  runs `ngspice -b` as a subprocess, parses the rawfile output, and returns
  structured data.
- **kitest-py** is a PyO3 binding so testbenches can be written in Python and
  run under pytest.

Running ngspice as a subprocess (not via its shared library FFI) is a deliberate
choice. The shared library has single-instance global state, which blocks
parallel test execution. The subprocess approach also keeps kitest
license-independent from ngspice's GPL.

## Assertions

All assertions are tolerance-aware. No exact-float comparison. The vocabulary so
far:

- `settles_to(target, tolerance)` for transient settling
- `overshoot(limit)` for transient overshoot
- `gain_db_at(frequency, expected, tolerance)` for AC gain
- `phase_deg_at(frequency, expected, tolerance)` for AC phase

## Current state

The full pipeline works end to end: KiCad export, ngspice simulation, and
assertion checking for DC, transient, and AC analyses. The walking skeleton (a
resistor divider test) passes in CI.

What is not built yet: full Python testbench API (the PyO3 binding currently
only exposes `version()`), static/connectivity assertions, boundary/sheet
slicing for sub-circuit isolation, behavioral MCU-in-the-loop via Renode, and a
Xyce backend.

## Relicensing

I originally licensed KiTest under AGPL-3.0. I switched to LGPL-3.0-or-later
because AGPL's network clause guards a hosted-service scenario that does not
apply to a local/CI tool, and it blocked corporate adoption. Under LGPL,
kitest's own code stays copyleft, but importing it to write testbenches
imposes nothing on downstream works.

## Repository

[github.com/georgesleen/kitest](https://github.com/georgesleen/kitest)
