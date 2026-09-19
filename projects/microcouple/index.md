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

Microcouple is a pair of small RP2040 boards that talk to each other over infrared. Each one carries an LED matrix, capacitive touch pads, USB-C, and LiPo charging. Mostly it was a PCB-design exercise: how much circuitry has to sit around an RP2040 before it's a usable board.

The schematic covers the RP2040, IR transmit circuitry and a 38 kHz receiver, an LED matrix, capacitive touch inputs, USB-C, LiPo charging, and — from revision 2 onward — a W25Q128 QSPI flash device.

That flash was the correction. On revision 1 I saw "SoC", assumed I could just place the RP2040 and go, and left off the external QSPI storage the chip actually boots from. Revision 2 adds the 128 Mbit W25Q128 and routes the QSPI bus properly. "System on chip" doesn't mean "system on board".

The hardware side is fairly complete — KiCad hierarchy, PCB layout, fabrication outputs, 3D model — but there is no firmware. `firmware/src/main.py` is empty. The IR protocol, touch pads, LED animations, and USB behavior were never written.

_Schematic (page 1):_

![Microcouple schematic](media/microcouple-schematic-p1.svg)

[github.com/georgesleen/microcouple](https://github.com/georgesleen/microcouple)
