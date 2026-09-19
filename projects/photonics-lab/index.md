---
title: "Photonics Lab"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "Bringing up a lossy silicon photonic frequency discriminator, on the way to a linewidth measurement."
thumbnail: "media/ref-photonic-chip-closeup.jpeg"
date: 2026-06-17
status: "active"
featured: false
tags: ["electrical", "simulation", "sensing"]
media:
  - media/ref-photonic-chip-closeup.jpeg
  - media/ref-photonic-chip-fiber-array.jpeg
  - media/measurement-setup-block-diagram.png
  - media/lab-photo.jpg
  - media/carrier_maps.png
  - media/neff_vs_V.png
---

# Photonics Lab

![Photonic chip closeup](media/ref-photonic-chip-closeup.jpeg)

I work in a silicon photonics research group at UBC through SiEPIC, bringing up an existing photonic chip as a laser frequency discriminator. The chip was designed by Pegah Tekieh, fabricated by AMF in a PSiN-on-SOI C-band process, and packaged by Jamal with a fiber array.

## How it works

The discriminator is an unbalanced Mach-Zehnder interferometer. Light splits between two arms, one arm has a long delay spiral, and a 2×2 MMI recombines them onto a balanced photodiode pair. At quadrature, laser frequency noise turns into a differential photocurrent. A PIN VOA on the short arm balances loss, and a heater tunes the interferometer phase to hold quadrature. The chip's designed delay is `1.90 ns`, which sets the conversion between phase noise and frequency noise.

![Measurement setup block diagram](media/measurement-setup-block-diagram.png)

## Current state

The chip works, but it's lossy. Path P1 → MZI-1 → PD3/PD4 measures about `35.5 dB` of loss and `11.1 dB` of fringe extinction. Earlier logbook entries said `5.5 dB` and `41 dB` — those were wrong by a factor of 1000 in the photocurrent reading, and I checked them frame-by-frame against the recorded video of the original bench display.

The readout electronics work out to a `100 kHz` band (Analog Discovery 3 at 200 kS/s), and I have a first frequency-noise spectrum. It's not yet a linewidth measurement. The run was taken at low light, the calibration used a transduction figure measured at a different power, and the band is well below where the beta line matters for the laser under test. Reading a linewidth off that plot would be wrong.

The current blocker is optical, not electrical: not enough light is reaching the photodiodes to run the linewidth measurement properly.

## Simulation

Alongside the bench work, I ported the group's PIN VOA simulation from Lumerical CHARGE/MODE to open-source tools — DEVSIM for the drift-diffusion carrier solve, femwell for the optical eigenmode. The model sweeps a lateral p++/n++ PIN junction on a 220 nm SOI rib waveguide from 0 to 4 V forward bias and pulls out the effective-index change from the injected free carriers.

![Carrier density maps](media/carrier_maps.png)

![Effective index vs. voltage](media/neff_vs_V.png)

[VOA simulation](https://github.com/georgesleen/frequency-discriminator-voa-simulation)
