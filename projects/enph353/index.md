---
title: "ML Based Robot Detective"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "Our ENPH 353 competition robot: imitation learning for driving, YOLO for reading the clue boards."
thumbnail: "media/map.png"
date: 2025-12-06
status: "complete"
featured: false
tags: ["robotics", "ml", "simulation", "sensing", "mechanical"]
media:
  - media/map.png
  - media/data-collection-gui.png
  - media/pedestrian-detection.png
  - media/yolo-ocr.png
  - media/yolo_training_data.png
  - media/runpod-quad-5090.png
  - media/clue-collection.png
  - media/main.pdf
---

# ENPH 353 - HTTP 418

![Simulated world map](media/map.png)

For the ENPH 353 final, Joshua Himmens and I built a robot in simulation that drove a course, read the blue clue boards, avoided pedestrians and a truck, and recovered from crashes, from a single onboard camera. Team name was HTTP 418. Josh mostly worked on driving; I mostly worked on OCR and imitation-learning training. Josh also came up with the idea of using YOLO for character recognition and designed a lot of the training setup, so the line between the two halves is fuzzier than it sounds.

## What we ended up with

We tried reinforcement learning first. Training took days per model, none of them drove well enough for competition, and we ran out of iteration time. What actually competed was an imitation-learning model exported to ONNX. The export also solved a Python-version fight — training could happen in a modern TensorFlow environment while inference ran in the older Python that ships with ROS, fast enough to steer on every camera frame.

The first OCR plan also failed. One YOLO model that read individual characters well was bad at locating the whole clue board in-frame. We ended up finding the boards with an HSV threshold on their very specific blue, cropping them, and sending the crop to a custom YOLO OCR model on Modal. The clue collector kept a histogram of everything read and submitted the mode instead of trusting any single frame.

![YOLO OCR output](media/yolo-ocr.png)

The rest of the ROS system worked around those two models — a pedestrian tracker that waited for someone to move and then stop or leave the frame (which also caught the truck), and a crash detector that respawned the robot in Gazebo when the image stopped changing. The PyQt GUI showed the camera, model overlays, and tracker state so we could poke at one node at a time.

![Pedestrian and vehicle detection](media/pedestrian-detection.png)

## Training

I did most of the character-model training, both the clue-board and the per-character classes, and a lot of the imitation-learning training. Models went into Weights & Biases so the runtime could pull whichever one was tagged for competition.

![Training dataset snapshot](media/yolo_training_data.png)

Local YOLO training was painfully slow, so we rented time on Runpod. The saved screenshot has four RTX 5090s, and the report records roughly `100 GiB` of GPU memory available.

![Runpod training instance](media/runpod-quad-5090.png)

The final OCR pipeline read every sign we drove past, but it was far too expensive to run inline. Moving it to an autoscaling Modal endpoint meant a two-second glimpse of a sign turned into twenty or thirty independent guesses for the histogram. The driving model was cheap by comparison, but it needed extra data collection for the corners of the course it kept losing.

![Control GUI and data collection](media/data-collection-gui.png)

![Clue aggregation](media/clue-collection.png)

- [Simulation and control](https://github.com/enph353-2025-team2/ENPH353_HTTP_418)
- [Vision training](https://github.com/enph353-2025-team2/yolo-vision)
- [Final report](https://github.com/enph353-2025-team2/final-report)
- [Music server](https://github.com/enph353-2025-team2/music-server)

<iframe width="100%" height="450" src="https://www.youtube.com/embed/4jBRHqV6Ss8" title="Robot operating video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<object data="media/main.pdf" type="application/pdf" width="100%" height="900px"></object>
