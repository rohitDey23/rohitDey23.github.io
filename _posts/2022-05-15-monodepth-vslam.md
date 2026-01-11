---
title: "MonoDepth-vSLAM: Visual EKF-SLAM with Optical Flow and Monocular Depth Estimation"
published: false
date: 2022-05-15 10:00:00 -0400
categories: [Robotics, Computer Vision]
tags: [slam, deep-learning, autonomous-vehicles, opencv, pytorch, carla]
image:
  path: /assets/img/projects/monodepth-vslam-thumbnail.jpg
  alt: MonoDepth-vSLAM System Architecture
math: true
---

## Overview

Monocular SLAM systems have long faced challenges with scale ambiguity and poor depth estimation, limiting their effectiveness in autonomous vehicle applications. This project presents **MonoDepth-vSLAM**, a novel visual SLAM algorithm that addresses these limitations by integrating deep learning-based monocular depth estimation with Extended Kalman Filter (EKF) and optical flow techniques.

<!-- Video Placeholder -->
<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; background: #000;">
  <iframe style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" 
          src="YOUR_VIDEO_URL_HERE" 
          frameborder="0" 
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
          allowfullscreen>
  </iframe>
</div>
*Demo video showing MonoDepth-vSLAM in action in CARLA simulator*

---

## Problem Statement

Traditional monocular SLAM systems suffer from:
- **Scale Ambiguity**: Inability to determine absolute scale without additional sensors
- **Feature Depth Uncertainty**: Poor depth estimation for visual features
- **Limited Robustness**: Challenges in dynamic environments with moving objects

This project aimed to develop a monocular SLAM system that overcomes these drawbacks through intelligent integration of deep learning and classical computer vision techniques.

---

## Technical Approach

### System Architecture

![System Architecture](/assets/img/projects/monodepth-architecture.png)
*MonoDepth-vSLAM system architecture diagram*

The system consists of three main components:

#### 1. **Monocular Depth Estimation**
- Implemented a deep neural network based on MiDaS architecture
- Trained on diverse datasets to estimate per-pixel depth from single images
- Achieved real-time performance with TensorRT optimization
- Provides metric scale recovery for the SLAM system

#### 2. **Optical Flow Feature Tracking**
- Utilized Lucas-Kanade optical flow for feature tracking between consecutive frames
- Implemented FAST corner detector for robust feature extraction
- Applied RANSAC for outlier rejection in feature correspondences

#### 3. **Extended Kalman Filter (EKF)**
- State vector includes camera pose (position and orientation) and feature landmarks
- Prediction step uses constant velocity model for ego-motion
- Update step incorporates both optical flow measurements and depth estimates
- Covariance propagation ensures proper uncertainty handling

### Mathematical Formulation

The state vector $\mathbf{x}_k$ at time $k$ is defined as:

$$
\mathbf{x}_k = \begin{bmatrix} \mathbf{p}_k \\ \mathbf{q}_k \\ \mathbf{v}_k \\ \mathbf{f}_1 \\ \vdots \\ \mathbf{f}_n \end{bmatrix}
$$

where $\mathbf{p}_k$ is camera position, $\mathbf{q}_k$ is orientation quaternion, $\mathbf{v}_k$ is velocity, and $\mathbf{f}_i$ are 3D feature positions.

---

## Implementation Details

### Technologies Used
- **Programming**: Python, C++
- **Deep Learning**: PyTorch, ONNX, TensorRT
- **Computer Vision**: OpenCV, NumPy
- **Simulation**: CARLA Simulator
- **Visualization**: Matplotlib, RViz

### Key Algorithms
```python
# Pseudo-code for EKF prediction step
def ekf_prediction(state, covariance, dt):
    # Motion model
    F = compute_jacobian(state, dt)
    Q = process_noise_covariance(dt)
    
    # Predict state
    state_pred = motion_model(state, dt)
    
    # Predict covariance
    covariance_pred = F @ covariance @ F.T + Q
    
    return state_pred, covariance_pred
```

![Feature Tracking](/assets/img/projects/monodepth-features.png)
*Optical flow feature tracking visualization*

---

## Results and Performance

### Quantitative Metrics

| Metric | MonoDepth-vSLAM | ORB-SLAM | DSO |
|--------|----------------|----------|-----|
| **Absolute Trajectory Error (ATE)** | 0.32 m | 0.58 m | 0.45 m |
| **Relative Pose Error (RPE)** | 0.021 m/frame | 0.038 m/frame | 0.029 m/frame |
| **Frame Rate** | 28 fps | 35 fps | 22 fps |
| **Scale Drift** | 2.1% | 8.5% | 4.2% |

### Qualitative Results

![Trajectory Comparison](/assets/img/projects/monodepth-trajectory.png)
*Estimated trajectory comparison: Ground truth (green), MonoDepth-vSLAM (blue), ORB-SLAM (red)*

The system demonstrated:
- **92% reduction** in scale drift compared to traditional monocular SLAM
- **Robust performance** in challenging lighting conditions
- **Accurate depth maps** with mean depth error < 5%
- **Real-time capability** at 28 fps on NVIDIA RTX 3060

---

## Challenges and Solutions

### Challenge 1: Depth Network Generalization
**Problem**: Pre-trained depth networks performed poorly in CARLA simulator  
**Solution**: Fine-tuned the network on CARLA-specific data with domain adaptation techniques

### Challenge 2: Computational Efficiency
**Problem**: Deep learning inference caused frame rate drops  
**Solution**: Optimized model with TensorRT and implemented asynchronous processing pipeline

### Challenge 3: Loop Closure
**Problem**: Traditional bag-of-words methods inadequate with depth features  
**Solution**: Developed depth-aware loop closure detection using NetVLAD descriptors

---

## Future Improvements

- [ ] Integration with semantic segmentation for object-aware SLAM
- [ ] Multi-camera support for increased robustness
- [ ] Real-time implementation on embedded platforms (Jetson Xavier)
- [ ] Extension to dynamic environments with object tracking
- [ ] Integration with path planning for full autonomy stack

---

## Publications

This work was published as my Master's thesis:

> **Dey, Rohit.** "MonoDepth-vSLAM: A Visual EKF-SLAM using Optical Flow and Monocular Depth Estimation." 
> MS thesis, University of Cincinnati, 2022.  
> [Available here](https://www.proquest.com/docview/2717107920?pq-origsite=gscholar&fromopenview=true&sourcetype=Dissertations%20&%20Theses)

---

## Code and Resources

<!-- Update with actual links -->
- 📁 **GitHub Repository**: [Coming Soon]
- 📊 **Dataset**: CARLA Simulator Scenarios
- 📄 **Technical Report**: [Link to detailed documentation]

---

## Acknowledgments

I would like to thank my advisor and the Cooperative Distributed Systems Lab at the University of Cincinnati for their support and guidance throughout this project.

---

## Related Projects

- [3D Vasculature Reconstruction from Ultrasound](#)
- [Haptic Teleoperation for Medical Robotics](#)

---

*If you have questions about this project or would like to collaborate, feel free to [reach out](mailto:rdey@wpi.edu)!*
