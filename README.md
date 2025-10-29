# 🤖 Visual Servoing Project

---

## 📘 Introduction

Robot simulation through a compact and realistic environment provides opportunities for robotics, computer vision (CV), and machine learning (ML) programmers to learn and enhance their skills using various robot models and maps.  

**Gazebo** and **ROS** combined provide these services for accomplishing numerous tasks on the **Construct** platform.  
- **Gazebo** handles the environment — maps, objects, and robot architectures.  
- **ROS (Robot Operating System)** manages robot control (mobile robots, robotic arms, etc.) using organized files called *packages*.

The aim of this project is to develop an **automatic visual servoing system** that monitors the movement of a robot with an **eye-in-hand camera** in a specific course on *The Construct* platform — **Vision Basics: Follow Line**.  
Project files and video are provided.

---

## 🎯 Project Challenges

- 🟡 Use the robot-mounted camera to **follow the map’s yellow line**.  
- 🅿️ Use the same camera to **find and stop at a parking spot**, marked with a specific pattern (QR code, barcode, or parking sign).

---

## 🧩 ROS Materials

To accomplish these tasks, several ROS components are required. These together form the package used for the visual servoing system.

### ROS Components Overview

- **Packages:**  
  A collection of resources for a specific task (scripts, launch files, maps, configuration files).  
  A package typically includes:
  - `launch/` — Contains `.launch` files to automate system startup.  
  - `scripts/` — Contains executable Python files for specific tasks.  
  - `package.xml` — Lists package dependencies.  
  - `CMakeLists.txt` — Defines build and compilation settings.

- **Nodes:**  
  Executable units (Python/C++) that communicate via topics. Nodes can publish or subscribe to share data such as sensor readings.

- **Topics:**  
  Channels that carry data between nodes.  
  Example: one node publishes camera data, another node processes and displays it.

- **Messages:**  
  Data structures sent via topics. They trigger actions in subscribed nodes.

> 🧠 *Figure 1 – A general concept of a ROS package.*

---

## 🧱 Gazebo Materials

Gazebo is used to spawn all task-related objects, robots, and maps — simulating the real-world scenario for testing robot behavior.

> 🧩 *Figure 2 – A general concept of Gazebo services.*

---

## 📂 Project Directory Structure

Below is the general layout of the project repository:

```

visual_servoing_project/
├── launch/
│   ├── follow_line.launch
│   ├── image_pose_estimation.launch
│   └── spawn_object_master.launch
│
├── scripts/
│   ├── conv_img.py
│   ├── line_follower.py
│   └── parking_pose_estimation.py
│
├── config/
│   └── camera_params.yaml
│
├── images/
│   ├── reference_object.png
│   └── results/
│       ├── centroid_tracking.png
│       ├── feature_matching.png
│       └── homography_estimation.png
│
├── package.xml
└── CMakeLists.txt

````

---

## 🚀 Quick Start

### 🧰 Prerequisites

Ensure your environment includes:
- **ROS Noetic** or **Melodic**
- **Gazebo 9+**
- The following ROS packages:
  ```bash
  sudo apt install ros-noetic-vision-opencv ros-noetic-cv-bridge ros-noetic-image-transport ros-noetic-cmd-vel-mux

## Python 3 with **OpenCV** installed:

  ```bash
  pip install opencv-python
  ```

---

## 🚗 Task #1 — Line Following

**Goal:**
Make the robot follow the map’s yellow line using its onboard camera.

### Packages Used

* [`camera`](http://wiki.ros.org/Sensors/Cameras)
* [`cmd_vel_mux`](http://wiki.ros.org/cmd_vel_mux)
* [`vision_opencv`](http://wiki.ros.org/vision_opencv)
* [`cv_bridge`](http://wiki.ros.org/cv_bridge)

---

### 🏁 Launch Instructions

#### Step 1 — Launch Simulation Environment

```bash
roslaunch visual_servoing_project follow_line.launch
```

#### Step 2 — Run Image Conversion Node

```bash
rosrun visual_servoing_project conv_img.py
```

#### Step 3 — Run Line Follower Node

```bash
rosrun visual_servoing_project line_follower.py
```

---

### 🧮 Image Processing Pipeline

1. **Subscribe** to `/camera/rgb/image_raw`
2. **Convert** ROS image → OpenCV format (`bgr8`)
3. **Crop** unnecessary areas (`y=260–440`, `x=1–640`)
4. **Convert** to HSV color space
5. **Mask** the yellow region (`[20,200,200]` to `[40,255,255]`)
6. **Find Centroid** using `cv2.moments()`
7. **Draw** centroid and output filtered images
8. **Compute Motion** via proportional control

   * Constant `linear.x`
   * Variable `angular.z` based on centroid offset
9. **Publish** movement to `/cmd_vel`

---

## 🅿️ Task #2 — Object Detection and Parking

**Goal:**
Use the robot’s camera to detect and approach a parking target marked with a specific pattern.

### 🗺️ Launch Instructions

#### Step 1 — Spawn Map and Objects

```bash
roslaunch visual_servoing_project spawn_object_master.launch
```

#### Step 2 — Initialize Pose Estimation Node

```bash
roslaunch visual_servoing_project image_pose_estimation.launch
```

---

### ⚙️ Processing Workflow

1. **Feature Extraction (SIFT)**

   * Detect scale- and rotation-invariant features from a reference image.
   * Store these for matching.

2. **Feature Matching**

   * Compare live camera frames to reference features.
   * Identify matched keypoints.

3. **Pose Estimation (Homography)**

   * Compute transformation matrix between reference and current frame.
   * Estimate position/orientation of target.

4. **Parking Control**

   * Move robot to align target in image center.
   * Advance forward until distance threshold is met.
   * Stop when parked.

---

## 🧾 Conclusions

We simulated visual servoing tasks using a **TurtleBot with an RGB camera** on *The Construct*.

1. **Task 1:** Line following — used OpenCV filtering and centroid-based proportional control.
2. **Task 2:** Object-based parking — used SIFT feature matching and homography estimation.

Despite challenges (low FPS, server disconnections, trial-based calibration), the system achieved reliable performance.

**Visual Servoing** is a powerful approach for robotic perception and control — applicable to complex tasks like **autonomous navigation** and **robot-assisted surgery**.

---

## 📚 References

1. [Cameras](http://wiki.ros.org/Sensors/Cameras)
2. [cmd_vel_mux](http://wiki.ros.org/cmd_vel_mux)
3. [vision_opencv](http://wiki.ros.org/vision_opencv)
4. [cv_bridge](http://wiki.ros.org/cv_bridge)
5. Lowe, D. G. (2004). *Distinctive Image Features from Scale-Invariant Keypoints.* *IJCV*, 60(2), 91–110.
6. Melissianos, V. D. (2019). *Classification of medical images with modern visual processing methods.* TEI of Crete.
7. Harris, C., & Stephens, M. (2013). *A Combined Corner and Edge Detector.*
8. Bay, H., Ess, A., Tuytelaars, T., & Van Gool, L. (2008). *Speeded-Up Robust Features (SURF).* *CVIU*, 110(3), 346–359.
9. [Using Homography for Pose Estimation in OpenCV](https://medium.com/analytics-vidhya/using-homography-for-pose-estimation-in-opencv-a7215f260fdd)

---



