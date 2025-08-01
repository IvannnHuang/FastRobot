# Path Planning and Execution – Embedded System with Real-World Localization

This project implements a fully autonomous robot system capable of navigating, mapping its surroundings, and localizing itself between waypoints using **Bayesian localization**, **Kalman-filtered distance sensing**, and **IMU-based orientation PID control**. The system runs on a custom-built mobile robot using an **Artemis microcontroller (Apollo3)** for real-time control and a **Python Jupyter Notebook** interface for path planning, localization, and BLE communication.

> Embedded systems project using Python & C++ + BLE control  
> Robot built by modifying a toy car and integrating custom electronics  
> [View Full Hardware and Software Report](https://ivannnhuang.github.io/)

---

## Robot Hardware Overview

The robot was constructed by retrofitting a toy car chassis with:

- **Artemis microcontroller** (RedBoard Artemis Nano)  
- **Two motor drivers**  
- **VL53L0X Time-of-Flight (ToF) sensor**  
- **MPU6050 IMU sensor**  
- **Bluetooth Low Energy (BLE)**
<img src="assets/img/portfolio/car_latyout.JPG" alt="Final Robot" width="400"/>

---

## Sensor Fusion and Filtering

To increase precision in movement and mapping, the ToF sensor’s noisy output was processed using a **1D Kalman filter**, implemented directly on the Artemis board. This improved:

- Distance measurement stability
- Control accuracy during forward motion
- Localization confidence by providing clean scan data:

<img src="assets/img/portfolio/kf_inter_mea.png" alt="Final Robot" width="400"/>

---

## Mapping and Localization

A full **localization pipeline** was implemented:

1. The robot rotates 360° using the servo-mounted ToF sensor, collecting 18 measurements at 20° increments.
2. This scan is transmitted via BLE to MATLAB.
3. A **Bayes filter** is applied in MATLAB using a known environment map.
4. The result is a probabilistic estimate of the robot’s pose `(x, y, θ)`.
5. This pose is used to guide the next navigation step, correcting for drift:
<img src="assets/img/portfolio/map_global_wall.png" alt="Final Robot" width="400"/>

---

## Path Planning and Execution

The core navigation routine is handled by a MATLAB function `navigate_to_target`, which performs the following:

1. **Turn to face the target waypoint**  
   - Uses `pid_orient_imu` with IMU feedback for angular correction
2. **Drive forward**  
   - Uses `pid_position_tof` with ToF sensor for distance control
3. **Reorient to 0°**  
   - Standardizes robot heading for the next localization phase

Each phase is controlled by loop counters and PID tuning, ensuring the robot does not stall or overshoot.

<img src="assets/img/portfolio/navig_traj.png" alt="Final Robot" width="400"/>

---

## Top-Level Controller

The robot repeatedly performs the following loop:

```text
1. Localize via 360° scan
2. Navigate to next waypoint
3. Reset heading to 0°
4. Re-localize
5. Repeat until all waypoints are visited
