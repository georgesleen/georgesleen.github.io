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

The 8-bit computer taught me what the pieces of a CPU do. Now I am trying the
same problem in SystemVerilog with RISC-V instead of my own instruction set. It
is nowhere near a complete RV32I core yet.

`JAL` and stores have execute paths. `OP-IMM` doesn't decode the ALU function
yet, so everything in that class behaves like `ADDI`. The branch immediate is
still a sign bit followed by zeros, so there is no working conditional branch.

The register file has a proper testbench for reset, the hard-wired `x0`, both
read ports, writes, and same-cycle read/write behavior. The full-system
testbench can load a hex program, dump waveforms, and watch memory-mapped print
and completion writes. It is a convenient way to run the few instructions that
exist, not broad CPU verification.

The next job is finishing the branch immediate and execute path.

[github.com/georgesleen/verilog-cpu](https://github.com/georgesleen/verilog-cpu)
