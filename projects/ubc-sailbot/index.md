---
title: "UBC Sailbot"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "Shared STM32 firmware, protocol work, and a rudder-controller investigation on Polaris."
thumbnail: "media/polaris-imu-pcb-orthographic.png"
date: 2026-08-16
status: "design-teams"
featured: false
tags: ["embedded", "firmware", "electrical"]
media:
  - media/polaris-imu-pcb-orthographic.png
  - media/gdb-debugging.png
  - media/uart-loopback-test.png
  - media/nmea-messages-printing.png
---

# UBC Sailbot

I joined UBC Sailbot in September 2025 and started on the shared communication
firmware. Every controller on the boat runs the same STM32U575 codebase, so a
change to a sensor parser or a protocol touches the rudder, the wingsail, the
power distribution board, and the sensor module all at once.

My first few weeks were mostly spent getting boards to talk. Lots of UART
loopback and I2C poking with an Analog Discovery 3, and GDB on the wind-sensor
driver.

![UART loopback test](media/uart-loopback-test.png)

![GDB debugging session](media/gdb-debugging.png)

## Communication firmware

The LCJ CV7 wind sensor talks NMEA0183, and I worked on the typed parser for it.
Instead of raw fixed-point integers with a comment explaining how to decode
them, the messages come out as structs that say what they are.

One bug I chased down there: the scheduler was publishing CAN message
`0x040 SAIL_WIND` after both the MWV and XDR sentences. XDR only carries
temperature, so every second message shipped stale wind data. Commit `f841995`
publishes it on MWV only.

![NMEA messages during bring-up](media/nmea-messages-printing.png)

I also worked on the COBS-framed link between PLRS-IMU and the rudder
controller, the protocol documentation, branch integration, and CI checks for
host tests and STM32 peripheral configuration. My larger heading-sensor work is
documented on the [PLRS-IMU page](/projects/plrs-imu/).

## Checking the rudder refactor

After four months of rudder tuning and refactoring landed from two parallel
branches, nobody could say whether the merged controller still behaved like the
one that had been flying. I built a differential harness and pushed the same 200
randomized trajectories, 10,000 samples, through both builds.

The shipped firmware defines `STRAIGHT_ONLY`. On that path the refactor matched
the old controller after two bug fixes. A tuning branch had accidentally added
the integral twice, so the integral term ran too fast. Some controller and
maneuver state was also uninitialized; that bug was already in the older code.

The other state-machine paths had changed on purpose and still had open bugs.
My conclusion was limited to the code we actually ship: `STRAIGHT_ONLY` stayed
equivalent apart from those two fixes.

## Repositories

- [Communication firmware](https://github.com/UBCSailbot/com-module-firmware)
- [PLRS-IMU](/projects/plrs-imu/)
