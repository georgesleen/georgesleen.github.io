---
title: "Microcouple"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "A pair of small RP2040 IR boards with an LED matrix, touch pads, USB-C, and LiPo charging."
thumbnail: "media/microcouple-image.jpg"
date: 2025-01-01
status: "complete"
featured: false
tags: [ "embedded", "firmware", "pcbs", "sensing", "electrical", "mechanical" ]
media:
  - media/microcouple-image.jpg
  - media/microcouple-schematic-p1.svg
---

# Microcouple

![Microcouple board](media/microcouple-image.jpg)

Microcouple was meant to be a pair of small RP2040 boards that talk over
infrared. Each one has its own LED matrix, capacitive touch pads, USB-C, and a
LiPo charger. I mostly used the project to learn how much circuitry has to
surround an RP2040 before it becomes a usable board.

The schematic includes:

- an RP2040;
- infrared transmit circuitry and a 38 kHz receiver path;
- an LED matrix and capacitive-touch inputs;
- USB-C input and data connections;
- LiPo charging and battery power; and
- on revision 2, a W25Q128 QSPI flash device.

I got that wrong the first time around. I saw "system on chip", assumed I could
just place the RP2040 and go, and left off the external QSPI flash it boots
from. Revision 2 adds a 128 Mbit W25Q128 and routes the QSPI bus properly. A
system on chip is not a system on a board.

The hardware side is fairly complete: KiCad hierarchy, PCB layout, fabrication
outputs, and a 3D model. The firmware is not. `firmware/src/main.py` is an empty
file, so the IR protocol, the touch pads, the LED animations, and the USB side
were never written or tested.

_Schematic, first page:_

![Microcouple schematic](media/microcouple-schematic-p1.svg)

## Repository

[github.com/georgesleen/microcouple](https://github.com/georgesleen/microcouple)
