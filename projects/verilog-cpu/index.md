---
title: "Verilog CPU"
author: "openai-codex/gpt-5.6-sol"
layout: base.njk
description: "An early RV32I subset in SystemVerilog. JAL, stores, ADDI, and a lot still missing."
thumbnail: "media/thumbnail.png"
date: 2025-09-01T12:00:00+08:00
status: "active"
tags: ["computer architecture", "pcbs", "electrical", "simulation"]
---

# Verilog CPU

The 8-bit computer taught me what pieces a CPU has. This is me trying again in SystemVerilog against a real instruction set — an RV32I subset in progress, nowhere near a complete core.

`JAL` and stores have real execute paths. The `OP-IMM` case doesn't decode the ALU function yet, so everything in that class currently behaves like `ADDI`. The B-type immediate is still a sign bit followed by zeros, so conditional branches don't work — that's the next architectural gap, not a small bug to file.

The register file has a Unity-style testbench: reset, hard-wired `x0`, both read ports, writes, and same-cycle read/write behavior. The full-system testbench can load a hex program, dump waveforms, and observe memory-mapped print and completion writes. It's a useful way to run the few instructions that exist; it isn't broad CPU verification.

Next thing to do is the branch decode and execute path.

[github.com/georgesleen/verilog-cpu](https://github.com/georgesleen/verilog-cpu)
