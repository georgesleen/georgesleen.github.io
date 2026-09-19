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

For the ENPH 353 final competition, Joshua Himmens and I had to make a simulated
robot drive a city course, read clue boards, avoid pedestrians and vehicles, and
recover from crashes using only its camera feed. We called the team HTTP 418.
Josh spent most of his time on driving. I spent most of mine training OCR and
imitation-learning models. Josh also proposed using YOLO for the characters and
designed much of the training scheme, so there was plenty of overlap.

## What survived to competition

We tried reinforcement learning first. Training took days per model, none of
them drove well enough for competition, and we ran out of time to keep
iterating. What we actually competed with was an imitation-learning model
exported to ONNX. The export also solved a practical problem: we could train in
a modern TensorFlow environment and still run inference inside the old Python
that ships with ROS, fast enough to steer on every camera frame.

The first OCR plan failed too. One YOLO model read isolated characters well but
was bad at spotting whole clue boards in the world. In the end we found the sign
borders with an HSV threshold on their very specific blue, cropped them, and
sent the crop to a custom YOLO OCR model running on Modal. The clue collector
kept a histogram of everything it read and submitted the most common answer
instead of trusting any single frame.

![YOLO OCR output](media/yolo-ocr.png)

The rest of the ROS system worked around those two models. A pedestrian tracker
waited for someone to move and then stop or leave the frame, and the same path
caught the truck. A crash detector noticed when the image stopped changing and
respawned the robot in Gazebo. Our PyQt GUI showed the camera, the model
overlays, the tracker state, and the data-collection controls, so we could poke
at one node at a time.

![Pedestrian and vehicle detection](media/pedestrian-detection.png)

## Training and integration

I built the training data and trained the character model, both the clue-board
and the per-character classes, and did a lot of the imitation-learning training
too. Models went into Weights & Biases, so the robot could pull whatever was
tagged for competition without us rebuilding anything.

![Training dataset snapshot](media/yolo_training_data.png)

Training YOLO locally was painfully slow, so we rented a Runpod machine. The
saved screenshot shows four RTX 5090s, and the report records roughly `100 GiB`
of available GPU memory. Getting the batches and artifact uploads to behave
across all four cards took a fair bit of the time we had hoped to save.

![Runpod training instance](media/runpod-quad-5090.png)

The final report says the OCR ended up very reliable, reading every sign we
drove past, but it was far too slow to run inline. Pushing it to an autoscaling
endpoint meant a two-second glimpse of a sign turned into twenty or thirty
guesses for the histogram. The driving model was cheap by comparison, but it
needed extra data collection for the parts of the course it kept failing.

![Control GUI and data collection](media/data-collection-gui.png)

![Clue aggregation](media/clue-collection.png)

## Project record

- [Simulation and control](https://github.com/enph353-2025-team2/ENPH353_HTTP_418)
- [Vision training](https://github.com/enph353-2025-team2/yolo-vision)
- [Final report](https://github.com/enph353-2025-team2/final-report)
- [Competition music server](https://github.com/enph353-2025-team2/music-server)

<iframe width="100%" height="450" src="https://www.youtube.com/embed/4jBRHqV6Ss8" title="Robot operating video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<object data="media/main.pdf" type="application/pdf" width="100%" height="900px"></object>
