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

PLRS-IMU is the heading sensor on Polaris, UBC Sailbot's autonomous boat. I expected most of the work to be the Kalman filter. Most of it has been sensors, GNSS bring-up, and deciding when the rudder should stop trusting the heading I'm handing it.

The firmware runs FreeRTOS on an RP2040. On the boat it's a Pico with i2c0 on GP16/17 (the pads the old MTi UART used to sit on); on my desk it's a Feather with i2c1 on A0/A1. Both pin sets live in `hardware_config.h`.

The IMU was originally an Xsens MTi-3. It died, so it's now a BNO085, and I wrote the SHTP/SH-2 protocol layer to talk to it. Once GNSS has constrained the magnetometer's heading offset well enough, a low-priority task saves that offset to flash so the next boot doesn't start from an arbitrary reference.

![Mag cal spin](media/mag-cal-spin.png)

## The filter

The EKF carries seven states: heading, roll, pitch, three axes of gyro bias, and one magnetometer heading offset.

Body-Z gyro integrated straight into heading is fine on flat water. At 20° of heel, the vertical axis of rotation isn't body-Z anymore, and the filter reads a slow yaw that isn't there. Roll and pitch as state, with the gyro mapped through the current attitude before integrating.

The BNO's magnetic yaw isn't true heading — declination, boat iron, and the ENU-to-compass sign flip all sit between them, and indoors that gap drifts. The mag measurement corrects `heading + offset` instead of heading directly. GNSS still owns absolute heading; the mag pins the sum.

The rudder link carries heading, roll, pitch, yaw rate, and a `heading_valid` flag. That flag needs the heading variance inside its threshold, pitch nowhere near the ZYX singularity, and a good BNO calibration status. If any of those goes bad, the rudder falls back to its own logic.

## The GNSS was fine; my diagnostic wasn't

The GNSS is a Septentrio mosaic-go H — dual antenna, produces a real compass heading from the baseline between the two.

Our first one was broken. Main tracked ~28 satellites in clear sky, aux tracked 0 to 1, no heading. It went back as an RMA. The hardware manual is explicit that ANT_2 is AC-coupled and unprotected; I think it died from hot-plugging the SMA with the 5 V bias live.

The second unit's aux tracked 3 to 6 satellites cleanly, but the attitude solve still failed. My diagnostic reported "0 common satellites", and I spent a while assuming we'd been shipped a second dead receiver.

My parser only counted MeasEpoch Type1 sub-blocks and ignored the nested Type2 sub-blocks, which is where the aux antenna reports most of its measurements. The two antennas actually had 26 common satellites the whole time. After the parser fix, heading locked at 241° and stayed within about 0.15° peak-to-peak while stationary.

## GNSS outages

With default tuning, the sim drops `heading_valid` about 16 seconds into an outage. My first framing was to call that "16 seconds of usable heading", which is wrong — that's just when the covariance grows past the rudder's 5° sigma gate. It tells you when the rudder gives up, not how accurate the heading was before that.

Usable coast time comes from how tightly the mag offset is pinned during the outage. Normally the offset floats loose so it can absorb magnetometer wander instead of dragging heading around. During an outage that same looseness lets confidence bleed off in ~16 seconds. Opt-in setting `q_offset_outage`: after GNSS has been gone longer than a short grace period, the offset random walk switches to a much smaller value.

Clean simulated magnetometer: heading held within 1° over a 5-minute outage, `heading_valid` stayed true the whole time. Moderate or indoor mag: heading held confidently 10 to 13 degrees off. The setting defaults off, safe to enable only once the boat's magnetics have been characterized.

![GNSS outage, offset pinned](media/sim-outage-hold.png)

![Same at 20° heel with a body-Y gyro bias](media/sim-heel-outage.png)

![Peak drift over a 30 s outage across the attitude envelope](media/sim-drift-sweep.png)

The Python harness runs the same firmware EKF through nanobind, so there isn't a second implementation quietly disagreeing about floating-point ordering. It's still a simulation result — I haven't reproduced the clean-mag hold on Polaris yet.

## The board

The PCB is my design in KiCad. RP2040, the IMU, a debug-probe header, and overvoltage protection on the inputs. It talks to the rudder controller over the COBS-framed serial link from the [com-module-firmware](/projects/ubc-sailbot/).

![PCB front](media/polaris-imu-pcb-front.png)

![PCB back](media/polaris-imu-pcb-back.png)

- Firmware and simulation: [github.com/UBCSailbot/PLRS-IMU](https://github.com/UBCSailbot/PLRS-IMU)
- PCB: [github.com/georgesleen/polaris-imu-pcb](https://github.com/georgesleen/polaris-imu-pcb)
