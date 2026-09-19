---
title: "Continuous Servo Control PCB"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "Control circuit for a servo with speed feedback and PI control."
thumbnail: "media/enph259-servo-front.png"
date: 2024-11-01
status: "complete"
featured: false
tags: [ "pcbs", "controls", "electrical", "sensing", "mechanical" ]
media:
  - media/enph259-servo-front.png
  - media/final-product.jpg
  - media/servo-schematic-p1.svg
  - media/servo-front-pcb-p1.svg
  - media/servo-lab-setup.png
---

# Continuous Servo Control PCB

For an ENPH 259 follow-on, I tried to control a motor without hiding the loop
inside a microcontroller. The board measures shaft motion digitally, converts
the count back to a voltage, and closes the loop with an analog PI controller.

The sensor is a slotted disk on the shaft. Its pulses feed a counter which is
latched immediately before a periodic reset, producing a sampled measure of
rotational speed. An R-2R ladder converts the latched binary value to a voltage.
That voltage is compared with the reference, and the resulting error passes
through the PI stage before driving the motor through MOSFETs.

The result is half digital and half analog. Counting and latching happen in
logic, while the R-2R conversion, error signal, and controller stay analog. I
designed the schematic and PCB in KiCad, added a binary speed readout, and built
the board and slotted-disk mount.

![Assembled board and servo](media/final-product.jpg)

The setup expects external `+5 V`, ground, `-5 V`, and a `5 Hz` clock. That
clock defines the counting window, so it is part of the measurement rather than
just a convenience for the logic.

![Lab setup](media/servo-lab-setup.png)

I assembled the circuit, but I didn't save a final step-response plot or any
other closed-loop measurement. I know how the loop was supposed to work; I
can't put an accuracy or stability number on the finished board.

## Design files

_PCB render:_

![Board render](media/enph259-servo-front.png)

_Schematic, first page:_

![Servo schematic](media/servo-schematic-p1.svg)

_PCB drawing, first page:_

![Servo PCB front](media/servo-front-pcb-p1.svg)

## Repository

[github.com/georgesleen/enph259-servo-pcb](https://github.com/georgesleen/enph259-servo-pcb)
