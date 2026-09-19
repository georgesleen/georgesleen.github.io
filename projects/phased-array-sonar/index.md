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

Joshua Himmens and I are working on the electronics for an ultrasonic array. We
have a main-board design, a TX/RX development board, some SPICE work, and a
shared KiCad library. We haven't built and tested the full sonar system.

I designed most of the power tree and the transmit H-bridge. Josh designed most
of the receive path, including the MEMS microphone preamplifier and the
individual receive channels.

## Power and transmit work

The board can accept USB-C or an XT60-connected source through a power mux and
ideal-diode path. From there, separate converter and regulator sheets generate
the positive, negative, digital, and analog rails needed by the transmit and
receive sections.

![Power-supply hierarchy](media/power-supply-hierarchy.png)

I designed that tree and spent the most time on the inverting buck-boost stage.
I also designed the H-bridge sheet for the transmit elements. There is still a
fairly obvious mismatch in the files: the H-bridge sheet says `20 kHz`, but the
selected Murata MA40S4S transducer is a `40 kHz` part. We need to fix that
before either frequency means anything on the bench.

## Receive path and tooling

Josh's receive design uses analog MEMS microphones, grouped preamplifiers, and
individual receive channels. The separate `rx_amp_sim` KiCad project lets him
work on the amplifier in SPICE; the real receive chain hasn't been built or
measured.

I combined the old main-board and TX/RX-development-board repositories into this
monorepo and worked on the shared KiCad setup. The script is now at
`sonar-library/setup-kicad.py`. It registers the shared symbols and footprints,
sets the path variables, and refuses to rewrite the config while KiCad is open.

![PCB layout](media/pcb-top.png)

Next we need to check every rail and get one transmit/receive channel working.
There is no array-control software or end-to-end ranging result yet.

## Repository

[github.com/fizzy-sonar/sonar-hardware](https://github.com/fizzy-sonar/sonar-hardware)
