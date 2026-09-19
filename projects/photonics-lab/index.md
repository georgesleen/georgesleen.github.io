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

I joined a UBC photonics lab to bring up an existing silicon-photonic frequency
discriminator. Pegah Tekieh designed the chip, AMF fabricated it in a
PSiN-on-SOI C-band process, and Jamal packaged it with a fiber array. I have been
working on the bench setup, measurement software, and debugging.

The discriminator is an unbalanced Mach-Zehnder interferometer. One arm contains
a long delay spiral; the other has a PIN attenuator for balancing the loss. The
two arms recombine on a balanced photodiode pair. At quadrature, laser frequency
noise becomes differential photocurrent. The chip's designed delay is `1.90 ns`,
which sets the conversion from phase noise to frequency noise.

![Measurement setup](media/measurement-setup-block-diagram.png)

## It makes fringes, but the light level is bad

We got fringes through the first interferometer, but the path has about
`35.5 dB` of loss and only `11.1 dB` of fringe extinction. My first logbook
entry said `5.5 dB` and `41 dB`. I had misread a microamp-scale current by a
factor of 1000, so I went back through the recorded instrument display and
corrected it.

That loss is now the main problem. There is enough signal to prove the chip
interferes, but not enough to treat every noise trace as laser noise. More recent
work brought up the balanced readout and an Analog Discovery 3 acquisition path
at `200 kS/s`, giving a `100 kHz` measurement band. The electronics can see that
band. The optical signal reaching them is the bottleneck.

I do have a first frequency-noise spectrum, but it isn't a linewidth
measurement. It was taken at low light, the calibration came from a
transduction figure measured at a different optical power, and the band stops
well below where the beta line matters for this laser. Reading a linewidth off
that plot would be wrong.

## Simulation work

I also ported the PIN attenuator simulation from the lab's Lumerical flow to
open-source tools: DEVSIM for the carrier transport and femwell for the optical
mode solve. The model sweeps the lateral PIN junction bias in a 220 nm SOI rib
waveguide and converts the injected carrier distribution into effective-index
change.

![Carrier density maps](media/carrier_maps.png)

![Effective index versus voltage](media/neff_vs_V.png)

Before I can measure linewidth, I need to recover and hold enough optical power
through the packaged input path. Then I can repeat the calibrated spectrum with
the full readout bandwidth.

## Repository

[VOA simulation](https://github.com/georgesleen/frequency-discriminator-voa-simulation)
