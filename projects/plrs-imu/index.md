---
title: "PLRS-IMU"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "Heading fusion for UBC Sailbot: a BNO085, dual-antenna GNSS, a 7-state EKF, and a lot of debugging."
thumbnail: "media/polaris-imu-pcb-orthographic.png"
date: 2026-08-16
status: "active"
featured: true
tags: ["embedded", "firmware", "pcbs", "controls", "sensing", "electrical", "simulation"]
media:
  - media/polaris-imu-pcb-orthographic.png
  - media/polaris-imu-pcb-front.png
  - media/polaris-imu-pcb-back.png
  - media/mag-cal-spin.png
  - media/sim-outage-hold.png
  - media/sim-heel-outage.png
  - media/sim-drift-sweep.png
---

# PLRS-IMU

![Polaris IMU PCB render](media/polaris-imu-pcb-orthographic.png)

PLRS-IMU is the heading system I work on for UBC Sailbot's autonomous boat,
Polaris. I expected the EKF to be the hard part. In practice, most of my time has
gone into dead sensors, GNSS bring-up, misleading diagnostics, and deciding when
the rudder should stop trusting the answer.

The firmware runs FreeRTOS and currently reads a BNO085 over I2C. The boat uses
an RP2040 Pico with `i2c0` on GP16/17. My bench Feather uses `i2c1` on GP26/27.
Keeping both mappings in `hardware_config.h` has saved me from debugging the
wrong pins more than once.

The BNO085 replaced an Xsens MTi-3 after that sensor died. I wrote the SHTP/SH-2
path and fitted it into the existing IMU, GNSS, fusion, rudder, and persistence
tasks. Once GNSS has pinned down the magnetic heading offset, a low-priority task
saves it to flash for the next boot.

![BNO085 calibration spin](media/mag-cal-spin.png)

## Why the filter has seven states

The EKF carries heading, roll, pitch, three body-frame gyro biases, and one
magnetic heading offset. Roll and pitch are in there because a heeled boat
doesn't measure yaw as pure body-Z rotation. The filter maps the gyro axes
through the current attitude before integrating heading, then uses GNSS for
absolute heading and the BNO085's magnetic yaw as a heading-plus-offset
measurement.

The rudder link sends heading, roll, pitch, yaw rate, and `heading_valid`. That
flag checks heading variance, the pitch limit around the Euler singularity, and
the BNO085 calibration status. Heading sigma and pitch are also available
separately in telemetry.

## The GNSS failure that was partly my diagnostic

The first Septentrio mosaic-go H tracked almost nothing on its auxiliary antenna
and went back as an RMA. The team's second unit initially looked broken in a
different way: my diagnostic printed zero common satellites and I treated that
as the receiver's failure.

The diagnosis was wrong. My parser counted only MeasEpoch Type1 sub-blocks, but
the auxiliary antenna put most observations in nested Type2 blocks. The same
capture actually had 26 common satellites. A direct hardware check then gave 26
valid heading epochs around `241°`, with about `0.15°` peak-to-peak variation
while stationary.

## GNSS outages

My first outage result needed a correction too. In simulation the filter drops
`heading_valid` after about 16 seconds, because heading sigma crosses the
rudder's 5° limit. That is when the rudder stops trusting the heading; it says
nothing about how accurate the heading was during those 16 seconds.

I added an opt-in outage setting that pins the learned magnetic offset after
GNSS has been absent for a short grace period. With clean simulated magnetic
data, that held heading within `1°` for a 300-second outage and stayed valid.
With moderate or indoor magnetic errors, the same setting remained confident
while being wrong by about 10–13°. It therefore defaults off until the boat's
magnetic environment has been measured.

![GNSS outage heading hold](media/sim-outage-hold.png)

![Heeled outage heading hold](media/sim-heel-outage.png)

![Outage drift sweep](media/sim-drift-sweep.png)

The Python simulator calls the firmware's C++ EKF through nanobind, so there is
no second filter slowly drifting away from the embedded one. The clean-mag
300-second hold is still a simulation. I haven't reproduced it on Polaris.

## Hardware files

These are renders of the IMU board design:

![PCB front](media/polaris-imu-pcb-front.png)

![PCB back](media/polaris-imu-pcb-back.png)

I have no PCB files showing an RP2354 production board. The hardware I can
confirm from the firmware is the Pico on the boat and the Feather on my bench.

## Repositories

- [Firmware and simulation](https://github.com/UBCSailbot/PLRS-IMU)
- [PCB design](https://github.com/georgesleen/polaris-imu-pcb)
