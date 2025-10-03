---
title: "'M.O.' – Autonomous Pet Retrieval Robot"
layout: project.njk
description: "Autonomous robot built for UBC Engineering Physics' 2025 Pet Rescue competition"
thumbnail: "media/thumbnail.png"
date: 2025-08-01
status: "complete"
featured: true
tags: [ "robotics", "embedded systems", "pcb design", "firmware" ]
media:
  - media/final-robot-side.png
  - media/complete-electronics.png
  - media/debris-traversal.mp4
---

# “M.O.” – Autonomous Pet Retrieval Robot

## Overview

ENPH 253, also known as *Robot Summer*, is a six-week design course where teams design and build an autonomous robot
from the ground up.

The 2025 challenge was to rescue seven pets (stuffed animals) from a burning building (an obstacle course filled with
debris) and return them to safety. The robot needed to complete the entire task without any human input.

Our robot, M.O. (named after the cleaning robot in *Wall-E*), combined multiple sensing and actuation systems: LiDAR
to detect the pets, toilet brushes mounted on arms to grab them, and a basket system that clipped to a zipline to return
them to the safe zone. The design emphasized robustness and reliability, since the course required line following,
object detection, and precision actuation in a noisy environment.

---

## My Contributions

I was the electrical lead on the team, responsible for circuit design and integration, as well as contributing to
the robot’s control firmware.

On the electrical side, I designed the custom H-bridge driver board for the drivetrain and handled the PCB design,
assembly, and troubleshooting for all major subsystems. Because the ESP32 we used had a limited number of pins, I also
designed around this constraint using multiplexing and multi-drop buses to maximize what we could do with the
available hardware.

On the firmware side, I worked on the concurrency and control logic of the robot. The ESP32 ran FreeRTOS, so I
implemented mutexes to safely handle concurrent access to I²C sensors. I also wrote the PID control loops that closed
the feedback on the drivetrain and actuators, which were essential for smooth navigation and reliable manipulation. As
the codebase grew, I helped maintain consistency and documentation so the team could continue building quickly without
things breaking.

---

## Technical Highlights

- **Controller:** ESP32 running FreeRTOS, coordinating sensing, navigation, and actuation
- **Sensors:** LiDAR for pet detection, IR reflectance sensors for line following
- **Actuation:** DC motors driven by custom H-bridge, servo release mechanism for the zipline basket
- **Electronics:** Isolated logic and motor grounds to reduce noise, multiplexed analog inputs to expand sensor capacity
- **System Design:** Modular PCBs for drivetrain and logic subsystems, enabling independent testing before integration

---

## GitHub

[github.com/enph253-2025-team5](https://github.com/enph253-2025-team5)

---

## Media
- *Final robot – side view*  
  ![Final robot side](media/final-robot-side.png)

- *Electronics assembly*  
  ![Complete electronics](media/complete-electronics.png)

- *Pet pickup test*  
  <video src="media/simple-pet-pickup.mp4" controls style="width:100%; height:auto; display:block;"></video>

- *Zipline basket return*  
  <video src="media/zipline-basket.mp4" controls style="width:100%; height:auto; display:block;"></video>

- *Line following demo*  
  <video src="media/first-line-following.mp4" controls style="width:100%; height:auto; display:block;"></video>

---

## Reflection

Building M.O. was an exercise in balancing ambition with reliability. We had only six weeks, which forced the design to
be simple enough to finish yet complex enough to handle the course.

I came away with a much stronger understanding of how embedded software and hardware interact in practice: ground noise
and I²C contention were just as real of challenges as line following or object detection. This project also underscored
how important clear roles, clean code, and documentation are when a team is moving quickly, small amounts of structure
made a huge difference in what we were able to deliver.
