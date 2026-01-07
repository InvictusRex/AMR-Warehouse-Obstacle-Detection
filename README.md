# AMR-Warehouse-Obstacle-Detection

This repository contains a vision-based system designed to improve navigation efficiency and safety for Autonomous Mobile Robots (AMRs) operating in warehouse environments. The system continuously monitors warehouse lanes using fixed, wall-mounted cameras and determines whether a lane is usable or blocked based on the presence of fallen boxes.

The primary objective is not robot navigation or localization, but **lane availability awareness**. By detecting obstacles at the infrastructure level and sharing lane states with the AMR fleet manager, robots can make better routing decisions without increasing onboard computational complexity.

## System Overview

Warehouses typically have predefined navigation lanes between shelves and along outer aisles. While these lanes are usually clear, operational realities such as fallen boxes can temporarily block them, leading to inefficient rerouting or unsafe conditions if not handled properly.

This system observes those lanes using static cameras installed on warehouse walls. Since the cameras do not move and the environment is largely structured, the perception problem becomes significantly simpler and more robust. The system detects box obstacles on the warehouse floor and determines whether a lane should be marked as FREE or BLOCKED until the obstruction is removed.

The output of the system is a continuous stream of lane state updates that can be consumed by an AMR fleet manager, global planner, or traffic control module.

## Camera Views and Lane Layout

The following images show sample views from the fixed, wall-mounted cameras installed in the warehouse.  
Each camera observes multiple navigation lanes, including inner lanes between shelves and outer lanes.

These views are used to define static lane regions of interest (ROIs) and to monitor the presence of box obstacles on the warehouse floor.

### Camera View 1

![Camera 01 View](Camera%20Angles/Camera_01.jpg)

### Camera View 2

![Camera View 2](Camera%20Angles/Camera_02.jpg)

### Camera View 3

![Camera View 3](Camera%20Angles/Camera_03.jpg)

### Camera View 4

![Camera View 4](Camera%20Angles/Camera_04.jpg)

## Perception Pipeline

Each camera feed is processed independently through a vision pipeline built around a YOLO-based object detector. The detector is trained to recognize a single class of interest: boxes that may obstruct robot movement.

Before runtime, the valid navigation lanes visible to each camera are defined as static polygonal regions of interest (ROIs). These lane definitions are stored as configuration files and loaded during execution. Because the cameras are fixed, these ROIs remain constant over time and do not require online calibration.

During operation, each incoming frame is passed through the YOLO model to detect boxes. For every detected box, the system checks whether it intersects with any lane ROI. A box is considered relevant only if it lies sufficiently within the lane region, which helps avoid false detections from shelf-level boxes or nearby clutter.

## Temporal Reasoning and Lane State

Lane usability is not determined from a single frame. Instead, the system applies temporal validation to ensure stability and reliability. If a box is detected within a lane consistently for a predefined number of consecutive frames, the lane is marked as BLOCKED. Similarly, once a blocked lane shows no box detections for a sufficient duration, it is marked as FREE again.

This temporal reasoning prevents false positives caused by brief occlusions, detection noise, or momentary events such as workers passing through the camera view.

In scenarios where a lane is visible from multiple cameras, a safety-first fusion strategy is applied. If any camera reports the lane as blocked, the lane is treated as blocked at the system level.

## Output and Integration

The system does not stream images or raw detections to robots. Instead, it publishes compact, structured lane state information that includes the lane identifier, its current state, the detected obstacle type, and a timestamp.

These lane state updates are intended to integrate seamlessly with higher-level systems such as fleet managers or planners. Robots can simply avoid blocked lanes in their navigation graphs and replan paths accordingly, without needing to understand the perception details.

## Design Principles

The system is intentionally designed to be simple, explainable, and scalable. It avoids depth estimation, SLAM, or robot-side perception dependencies. All intelligence resides at the infrastructure level, allowing a single deployment to support many robots simultaneously.

This architecture makes the system well-suited for real-world warehouse deployments where reliability, debuggability, and ease of maintenance are critical.
