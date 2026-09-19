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

An ENPH 259 follow-on: close a motor control loop without a microcontroller in the middle. The board measures shaft motion digitally, converts the count back into a voltage, and closes the loop with an analog PI controller and a MOSFET H-bridge driver.

The sensor is a slotted disk on the shaft. Pulses feed a counter that's latched right before a periodic reset, giving a sampled measure of rotational speed. An R-2R ladder converts the latched binary value to a voltage; that voltage is compared to a reference, and the error goes through the PI stage before driving the motor. Digital counting, analog control, one board. The system needs external `+5 V`, ground, `-5 V`, and a `5 Hz` clock, since the clock sets the counting window.

![Assembled board and servo](media/final-product.jpg)

![Lab setup](media/servo-lab-setup.png)

I built and powered up the board, but I didn't save a step response or any other closed-loop measurement, so I can't put a real accuracy number on it. What I have is a board whose stages all produce the expected signals on the bench.

_Board render:_

![Board render](media/enph259-servo-front.png)

_Schematic (page 1):_

![Servo schematic](media/servo-schematic-p1.svg)

_PCB front (page 1):_

![Servo PCB front](media/servo-front-pcb-p1.svg)

[github.com/georgesleen/enph259-servo-pcb](https://github.com/georgesleen/enph259-servo-pcb)
