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

## Overview

KiTest brings automated testing to analog and mixed-signal circuit design. The
idea is simple: a schematic captures the design, but nowhere does it capture
the intended behavior. What voltage should appear at this node? How fast should
the output settle? What is the gain at 10 kHz? Today, engineers verify these
properties by manually building SPICE test setups every time they change the
circuit. KiTest makes those tests persistent, one-command re-runnable artifacts
that live next to the design files.

The framing I like to use: **DRC checks manufacturability, ERC checks wiring,
KiTest checks intent.**

## How It Works

The pipeline has four stages:

1. Export a KiCad schematic to a SPICE netlist via `kicad-cli`
2. Inject test-side stimulus (DC supplies, AC sources) into the netlist
3. Run an analysis (operating point, transient, or AC sweep) through ngspice
4. Parse the results and check them against assertions

A test for a resistor divider looks like this: export the schematic, inject a
5 V supply, run an operating point analysis, and assert that the output node
settles to 2.5 V within 1% tolerance.

## Architecture

The core is written in Rust (edition 2024), structured as a Cargo workspace
with two crates:

- **kitest** contains the engine: a `Backend` trait with three methods
  (`run_op`, `run_tran`, `run_ac`), each returning a typed result. The ngspice
  backend runs `ngspice -b` as a subprocess, parses the rawfile output, and
  returns structured data.
- **kitest-py** is a PyO3 binding so that testbenches can be written in Python
  and run under pytest.

Running ngspice as a subprocess (rather than via its shared library FFI) is a
deliberate choice. The shared library has single-instance global state, which
blocks parallel test execution. The subprocess approach also keeps kitest
license-independent from ngspice's GPL dependencies.

## Assertions

All assertions are tolerance-aware. No exact-float comparison is ever used.
The assertion vocabulary so far:

- `settles_to(target, tolerance)` for transient settling
- `overshoot(limit)` for transient overshoot
- `gain_db_at(frequency, expected, tolerance)` for AC gain
- `phase_deg_at(frequency, expected, tolerance)` for AC phase

Tolerances support both absolute and percentage forms.

## Stimulus Injection

KiTest works with netlist "bodies" (no trailing `.end`). The `kicad-cli` export
wrapper strips the terminator, and the engine wraps the body in a `.control`
block with the analysis directive. Injected voltage sources use the prefix `Vkt`
(e.g. `Vkt1`, `Vkt2`) to avoid colliding with the design's own sources.

## Current State

KiTest is in active early development. The full pipeline works end to end:
KiCad export, ngspice simulation, and assertion checking for DC, transient,
and AC analyses. The walking skeleton (a resistor divider test) passes in CI.

What is not built yet:

- Full Python testbench API (the PyO3 binding currently exposes only `version()`)
- Static/connectivity assertions (no-SPICE tier)
- Boundary/sheet slicing for testing sub-circuits in isolation
- Behavioral MCU-in-the-loop via Renode
- Xyce backend

## Development Environment

The `flake.nix` provides the complete toolchain: Rust, ngspice, Python, uv,
and `kicad-small`. CI runs the same Nix shell (`nix develop -c make fmt-check
lint test`), so local and CI environments are identical.

## Relicensing

I originally licensed KiTest under AGPL-3.0. I switched to LGPL-3.0-or-later
because AGPL's network clause guards a hosted-service scenario that does not
apply to a local/CI tool, and it blocked corporate adoption. Under LGPL,
kitest's own code stays copyleft, but importing it to write testbenches imposes
nothing on downstream works.

## Repository

[github.com/georgesleen/kitest](https://github.com/georgesleen/kitest)
