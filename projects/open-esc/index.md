---
title: "Open ESC"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "An open-source electronic speed controller with firmware written in Rust."
thumbnail: "media/thumbnail.png"
date: 2025-09-01
status: "active"
featured: false
tags: [ "power", "electrical", "firmware", "embedded", "pcbs", "mechanical" ]
media:
  - media/bootstrap-high-side_2025-09-20.png
  - media/working-commutation-phase-ab_2025_09-20.png
  - media/complementary-pwm-with-deadtime.png
  - media/inverter-gate-voltages_2025-10-01.png
  - media/rust-code.png
  - media/dead-mosfets_2025-09-20.jpg
---

# Open ESC

I had used plenty of brushless motors without really knowing what happened
inside the ESC. For this project I decided to build the power stage myself and
write the controller in Rust on an RP2040.

## The first switching hardware failed

My first high-side supply approach could drive LEDs and produce phase waveforms,
but repeated switching killed the high-side MOSFETs.

![Failed MOSFETs](media/dead-mosfets_2025-09-20.jpg)

I replaced that path with a bootstrap drive and iterated on the capacitor
values. Moving from the initial `10 µF` choice to `100 nF` produced a usable
high-side transition in the bench experiments.

![Bootstrap high-side waveform](media/bootstrap-high-side_2025-09-20.png)

The next problem was shoot-through. I added complementary high-side and low-side
PWM with a dead-time offset, then checked both gate signals on the scope. This
trace shows the gap between the transitions. There was no motor under load for
this test.

![Complementary PWM with dead time](media/complementary-pwm-with-deadtime.png)

The phase-to-phase bench waveform showed the commutation electronics switching
between phases:

![Phase-to-phase commutation](media/working-commutation-phase-ab_2025_09-20.png)

<video src="media/bootstrap-led-commutation_2025-09-20.mp4" controls style="width:100%; height:auto; display:block;"></video>

## Firmware experiments

The firmware uses Embassy on the RP2040. It has a three-half-bridge driver,
complementary PWM, dead-time calculation, and a six-step trapezoidal commutation
table. I also experimented with sinusoidal commutation. Neither path has
sensorless feedback yet.

![Rust firmware](media/rust-code.png)

![Inverter gate voltages](media/inverter-gate-voltages_2025-10-01.png)

Back-EMF zero-crossing, sensorless startup, and closed-loop timing are all still
missing, and I don't have a saved test of this controller turning a BLDC motor
under load. So far I have only driven and measured gate and phase waveforms on
the bench.

## Repositories

- [Hardware](https://github.com/georgesleen/open-esc-hardware)
- [Firmware](https://github.com/georgesleen/open-esc-firmware)
