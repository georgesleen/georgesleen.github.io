---
title: "Phased Array Sonar"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "Power, transmit, and receive electronics for an ultrasonic array, still in design."
thumbnail: "media/schematic-toplevel.png"
date: 2026-04-08
status: "active"
featured: false
tags: ["pcbs", "electrical", "power", "simulation"]
media:
  - media/schematic-toplevel.png
  - media/power-supply-hierarchy.png
  - media/pcb-top.png
---

# Phased Array Sonar

![Top-level schematic](media/schematic-toplevel.png)

Joshua Himmens and I are building the electronics for an ultrasonic sonar array. There's a main board design, a TX/RX dev board, some SPICE work on the receive amplifier, and a shared KiCad library. There is not a completed and tested phased-array sonar.

I designed the power tree and the transmit-side H-bridge. Josh designed most of the receive path — the MEMS microphone frontend, the per-channel preamps, and the receive amplifier chain.

## Power

The board can run from USB-C or an XT60 (battery or bench supply) through a power mux with ideal-diode ORing. From there the tree derives every rail the rest of the board needs — 5 V to 12 V boost (TPS55340), an inverting buck-boost for the negative rail (TPS63700), an adjustable 3.3 V buck for digital, a -10 V LDO (LM337), a 12 V to 10 V LDO, and a 2.75 V LDO.

![Power supply hierarchy](media/power-supply-hierarchy.png)

The inverting buck-boost was the hardest single stage. I still keep `inverting_buck_boost_OLD.kicad_sch` around because at one point my commit message on that stage was "Do some more questionable math for the inverting buck boost", and I want the reference.

The transmit H-bridge sheet has an unresolved detail. It's labelled `20 kHz`, but the Murata MA40S4S transducer is a `40 kHz` part. That mismatch needs fixing before either number means anything on the bench.

## Receive and tooling

Josh's receive design uses SPV0142LR5H-1 MEMS microphones feeding a multi-stage amplifier, with a separate `rx_amp_sim/` KiCad project for iterating on the amplifier in SPICE. The receive chain itself hasn't been built and characterized.

I merged the old `sonar-v1-pcb` and `tx-rx-dev-board` repos into this monorepo and wrote the shared library tooling. `sonar-library/setup-kicad.py` (PEP 723 inline metadata) registers the library with KiCad's global tables and sets the path variables, and refuses to run while KiCad is open so it doesn't clobber the config.

![PCB layout](media/pcb-top.png)

Next real milestone is bringing up the rails and one transmit/receive channel. There is no array-control software or end-to-end ranging result yet.

[github.com/fizzy-sonar/sonar-hardware](https://github.com/fizzy-sonar/sonar-hardware)
