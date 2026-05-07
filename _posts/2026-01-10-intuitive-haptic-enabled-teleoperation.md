---
title: "Intuitive Haptic-Enabled Teleoperation for Neuro-Interventional Robotics"
date: 2024-08-20 14:30:00 -0400
categories: [Medical Robotics, Control Systems]
tags: [ros2, micro-ros, haptics, foc, esp32, surgical-robotics]
image:
  path: /assets/img/TIR/TIR_Thumbnail.jpg
  alt: Haptic Teleoperation System Setup
math: true
---

## **Objective**
Minimally invasive neuro-interventional procedures typically require surgeons to work in close proximity to X-ray fluoroscopy, leading to significant radiation exposure and physical fatigue. Furthermore, the manual manipulation of catheters and guidewires lacks the precise force feedback necessary for safe navigation through tortuous vasculature. This project aims to develop a high-fidelity, teleoperated robotic platform that removes the surgeon from the radiation field while restoring critical tactile sensation through active haptic feedback.

## **Methodology**

The system employs a master-slave architecture designed for transparency and low-latency control. The core innovation lies in the integration of modern embedded protocols with robust control theory to create a seamless user experience.

### **Control Architecture**
The control framework is built upon **ROS2** (Robot Operating System 2) for high-level communication and **micro-ROS** for real-time embedded control.
* **Master Console (Input):** The surgeon manipulates a custom haptic device tracked by an Electromagnetic (EMT) sensor. A "Clamping Detector" monitors the operator's grip to engage or disengage the system safely.
* **Slave Manipulator (Output):** The robotic manipulator is driven by **ESP32** microcontrollers. These execute Field Oriented Control (FOC) algorithms to drive the Brushless DC (BLDC) motors with high torque density and smooth motion.
* **Communication:** A high-speed WiFi link using the UDP protocol connects the master and slave, ensuring deterministic data transfer with minimal latency.

![Control System Block Diagram](/assets/img/TIR/haptic_contorl_diagram.png)
*Figure 1: Bilateral teleoperation control scheme. The system utilizes a PI filter to compensate for position error ($e_{\phi}$, $e_{\theta}$) and transmits torque commands ($\hat{\tau}$) over WiFi. The FOC current loop ($I_q$) on the slave side provides real-time force feedback to the haptic module.*

### **Haptic Feedback Loop**
As shown in the control diagram above, the system implements a bilateral control loop. The interaction forces between the guidewire and the vascular walls are estimated via the $I_q$ (quadrature current) component of the slave motors. This current is proportional to the torque and is fed back to the master console's haptic drives, allowing the surgeon to "feel" the resistance inside the patient's anatomy in real-time.

### **System Demonstration**
The demonstration below shows the master console in operation. The operator uses the handheld haptic interface to drive the guidewire, with the system translating natural hand motions into precise robotic insertion and rotation.

![Haptic Console Demo](/assets/img/TIR/Working_setup_demo.gif)
*Figure 2: Demonstration of the haptic input console controlling the slave manipulator.*

### **Guidewire Navigation Demo**

![Guidewire Navigation](/assets/img/TIR/guidewire_navigations.gif)
*Figure 3: Guidewire navigation through vascular phantom demonstrating precise control and haptic feedback.*

## **Key Achievements**

* **Radiation Safety:** Successfully decoupled the surgeon from the X-ray source, enabling remote operation from a radiation-shielded cockpit.
* **Clinical Efficiency:** Reduced simulated procedure time by **26%** compared to manual intervention due to improved ergonomic handling and intuitive controls.
* **High Precision:** Achieved **sub-millimeter accuracy** in guidewire positioning and sub-degree precision in rotation.
* **Real-Time Haptics:** Validated a stable haptic loop with low latency (<20ms), allowing operators to detect vessel wall collisions and friction.
* **Robust Connectivity:** Implemented a fault-tolerant ROS2/micro-ROS communication pipeline on ESP32 hardware capable of handling packet loss without system instability.

## **Conclusion**
This research presents a viable pathway for the next generation of surgical robotics. By combining the cost-effectiveness and performance of ESP32 microcontrollers with the robustness of the ROS2 ecosystem, we have created a platform that is not only precise and safe but also accessible. The integration of haptic feedback addresses a critical gap in current robotic surgery, promising better patient outcomes and improved surgeon well-being.

![Haptic Module in Action](/assets/img/TIR/PXL_20251217_221627038.gif)
*Figure 4: Haptic Module in Action.*
