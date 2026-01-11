---
title: "Haptic-Enabled Teleoperation Console for Neuro-Interventional Robotic Platform"
published: false
date: 2024-08-20 14:30:00 -0400
categories: [Medical Robotics, Teleoperation]
tags: [medical-robotics, haptics, ros2, control-systems, bldc-motors, micro-ros]
image:
  path: /assets/img/projects/haptic-robot-thumbnail.jpg
  alt: Haptic Teleoperation System for Medical Robotics
math: true
---

## Overview

Neuro-interventional procedures require extreme precision and dexterity when navigating guidewires and catheters through complex vasculature. This project presents an **intuitive haptic-enabled teleoperation console** that allows surgeons to perform these delicate procedures with enhanced control and tactile feedback, reducing radiation exposure and improving patient outcomes.

<!-- Video Placeholder -->
<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; background: #000;">
  <iframe style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" 
          src="YOUR_VIDEO_URL_HERE" 
          frameborder="0" 
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
          allowfullscreen>
  </iframe>
</div>
*Live demonstration of the teleoperation system in surgical simulation*

---

## Motivation

### Clinical Challenges
- **Radiation Exposure**: Surgeons are exposed to X-ray radiation during fluoroscopy-guided procedures
- **Physical Strain**: Long procedures cause surgeon fatigue and ergonomic issues
- **Limited Tactile Feedback**: Current robotic systems lack adequate force feedback
- **Steep Learning Curve**: Existing systems are complex and non-intuitive

### Our Solution
A teleoperated robotic platform that provides:
- ✅ Intuitive haptic interface mimicking natural surgical movements
- ✅ Real-time force feedback from guidewire-tissue interactions
- ✅ Remote operation capability for reduced radiation exposure
- ✅ Precise motion control with sub-millimeter accuracy

---

## System Architecture

![System Architecture](/assets/img/projects/haptic-system-architecture.png)
*Complete system architecture showing master-slave teleoperation setup*

### Key Components

#### 1. **Master Console (Operator Side)**
- Custom-designed haptic interface device
- 6-DOF force/torque sensor for input
- Electromagnetic tracking sensors for position sensing
- Ergonomic design based on surgeon feedback

#### 2. **Slave Robot (Patient Side)**
- Modular catheter/guidewire manipulation unit
- Multiple BLDC motors for precise motion control
- Force sensors for measuring tool-tissue interaction
- Compact design for OR integration

#### 3. **Control System**
- ROS2 framework for real-time communication
- micro-ROS for embedded motor controllers
- Bilateral teleoperation control algorithms
- Safety monitoring and emergency stop mechanisms

---

## Technical Implementation

### Field Oriented Control (FOC) for BLDC Motors

Implemented advanced motor control for precise catheter manipulation:

```cpp
// Simplified FOC implementation
void foc_control_loop() {
    // Read encoder position and velocity
    current_angle = read_encoder();
    current_velocity = compute_velocity();
    
    // Clarke transformation (3-phase to 2-phase)
    alpha = (2.0/3.0) * (Ia - 0.5*Ib - 0.5*Ic);
    beta = (2.0/3.0) * (sqrt(3)/2.0) * (Ib - Ic);
    
    // Park transformation (stationary to rotating frame)
    Id = alpha * cos(theta) + beta * sin(theta);
    Iq = -alpha * sin(theta) + beta * cos(theta);
    
    // PI controllers for current control
    Vd = pid_d.compute(Id_ref - Id);
    Vq = pid_q.compute(Iq_ref - Iq);
    
    // Inverse Park and Clarke for PWM generation
    apply_voltage(Vd, Vq, theta);
}
```

![FOC Control Loop](/assets/img/projects/haptic-foc-diagram.png)
*Field Oriented Control implementation for smooth motor operation*

### ROS2 Real-Time Motion Mapping

Developed custom ROS2 nodes for seamless integration:

```python
class EMTMotionMapper(Node):
    def __init__(self):
        super().__init__('emt_motion_mapper')
        
        # Subscriber for EMT sensor data
        self.emt_sub = self.create_subscription(
            PoseStamped, '/emt/pose', self.emt_callback, 10)
        
        # Publisher for robot commands
        self.cmd_pub = self.create_publisher(
            TwistStamped, '/robot/cmd_vel', 10)
        
        # Motion scaling and filtering
        self.motion_scaler = MotionScaler(scale_factor=0.5)
        self.filter = KalmanFilter()
    
    def emt_callback(self, msg):
        # Filter and scale operator motion
        filtered_pose = self.filter.update(msg.pose)
        scaled_motion = self.motion_scaler.scale(filtered_pose)
        
        # Publish to robot
        cmd = self.compute_velocity_command(scaled_motion)
        self.cmd_pub.publish(cmd)
```

### Electromagnetic Tracking Integration

![EMT Sensor Setup](/assets/img/projects/haptic-emt-sensors.png)
*Electromagnetic tracking sensor placement on the master console*

**Key Features:**
- 5-DOF position tracking with < 1mm accuracy
- Real-time pose estimation at 240 Hz
- Compensation for electromagnetic interference
- Workspace mapping and boundary enforcement

---

## Bilateral Teleoperation Control

Implemented a force-reflecting bilateral control scheme:

### Control Architecture

The system uses a 4-channel architecture with:
- **Position channel**: Master $\rightarrow$ Slave
- **Force channel**: Slave $\rightarrow$ Master
- **Transparency**: Operator feels actual forces
- **Stability**: Guaranteed through passivity

### Mathematical Model

The bilateral teleoperation dynamics:

$$
\begin{align}
M_m \ddot{x}_m + B_m \dot{x}_m &= F_h - F_{m} \\
M_s \ddot{x}_s + B_s \dot{x}_s &= F_s - F_e
\end{align}
$$

where:
- $x_m, x_s$ are master and slave positions
- $F_h$ is human operator force
- $F_e$ is environment (tissue) force
- $F_m, F_s$ are control forces

![Control Block Diagram](/assets/img/projects/haptic-control-diagram.png)
*Bilateral teleoperation control block diagram*

---

## Hardware Design and Prototyping

### CAD Models

![CAD Assembly](/assets/img/projects/haptic-cad-assembly.png)
*Complete CAD assembly of the robotic platform*

### Mechanical Design Highlights:
- **Lightweight construction**: Carbon fiber and aluminum alloy
- **Modular design**: Easy maintenance and component replacement
- **Sterilization-compatible**: Medical-grade materials
- **Compact footprint**: 300mm × 200mm × 150mm

### Electronics Integration

![Electronics Layout](/assets/img/projects/haptic-electronics.png)
*Custom PCB design for motor control and sensor integration*

**Key Specifications:**
- STM32H7 microcontroller for real-time control
- Custom BLDC driver boards with FOC
- Isolated CAN bus for motor communication
- Safety-rated emergency stop circuit

---

## Testing and Validation

### Benchtop Testing

Conducted extensive validation in laboratory settings:

| Test Parameter | Target | Achieved | Status |
|----------------|---------|----------|--------|
| **Position Accuracy** | ± 0.5 mm | ± 0.3 mm | ✅ Pass |
| **Force Feedback Latency** | < 20 ms | 15 ms | ✅ Pass |
| **System Bandwidth** | > 50 Hz | 68 Hz | ✅ Pass |
| **Trajectory Tracking Error** | < 1 mm | 0.7 mm | ✅ Pass |

### Phantom Testing

![Phantom Setup](/assets/img/projects/haptic-phantom-test.png)
*Vascular phantom testing setup with fluoroscopy*

Performed trials on anatomically accurate vascular phantoms:
- ✅ Successfully navigated complex vessel geometries
- ✅ Realistic force feedback during vessel interaction
- ✅ Reduced procedure time by 23% compared to manual operation
- ✅ Zero tool slippage or positioning errors

### User Studies

Preliminary evaluation with 5 interventional radiologists:
- **Ease of Use**: 8.2/10 average rating
- **Haptic Quality**: 7.8/10 average rating
- **Learning Curve**: "Intuitive within 30 minutes"
- **Clinical Potential**: 100% positive feedback

---

## Safety Features

Critical safety mechanisms implemented:

1. **Hardware E-Stop**: Redundant emergency stop circuits
2. **Virtual Fixtures**: Software boundaries prevent unsafe motions
3. **Force Limiting**: Automatic force clamping to safe thresholds
4. **Watchdog Timers**: System reset on communication loss
5. **Fail-Safe Modes**: Graceful degradation on component failure

---

## Challenges Overcome

### 1. Wireless Communication Reliability
**Challenge**: Maintaining real-time performance with wireless micro-ROS  
**Solution**: Implemented DDS QoS policies with deadline and liveliness monitoring

### 2. Motor Cogging and Vibration
**Challenge**: BLDC motors caused unwanted haptic vibrations  
**Solution**: Optimized FOC with sinusoidal commutation and vibration damping algorithms

### 3. EMT Sensor Interference
**Challenge**: Metal components distorted EMT sensor readings  
**Solution**: Calibration routine and real-time compensation using Kalman filtering

### 4. Force Sensor Drift
**Challenge**: Long-term drift in force measurements  
**Solution**: Periodic auto-zeroing and temperature compensation

---

## Current Status and Future Work

### Current Status (as of August 2024)
- ✅ Prototype V2 completed and functional
- ✅ Benchtop and phantom testing completed
- 🔄 Preparing for pre-clinical validation
- 🔄 Manuscript in preparation for IEEE ICRA 2025

### Future Directions
- [ ] Miniaturization for 3mm diameter catheters
- [ ] Integration with real-time imaging (fluoroscopy, ultrasound)
- [ ] Machine learning for autonomous navigation assistance
- [ ] Multi-tool manipulation capability
- [ ] Clinical trials at partner hospitals

---

## Technologies and Skills Demonstrated

**Robotics**: ROS2, micro-ROS, motion planning, kinematics  
**Control Systems**: PID control, FOC, bilateral teleoperation, Kalman filtering  
**Embedded Systems**: STM32, real-time programming, CAN bus, motor drivers  
**Mechanical Design**: SolidWorks, FEA, rapid prototyping, 3D printing  
**Sensors**: Force/torque sensors, EMT tracking, encoders  
**Software**: C++, Python, real-time Linux, DDS middleware

---

## Team and Acknowledgments

This project is being developed at the **Medical and Manufacturing Innovation Lab (MedMaIn)** at Worcester Polytechnic Institute under the guidance of Prof. [Advisor Name].

**Team Members:**
- Rohit Dey (Lead - Control Systems & Software)
- [Team Member 2] (Mechanical Design)
- [Team Member 3] (Electronics)

**Funding**: This work is supported by [Grant/Funding Source]

---

## Publications and Presentations

- Conference paper submitted to IEEE ICRA 2025
- Poster presentation at WPI Research Symposium 2024
- Live demo at [Medical Robotics Conference]

---

## Media and Press

<!-- Placeholder for media coverage -->
- [University Press Release](#)
- [Local News Feature](#)
- [Research Highlight Article](#)

---

## Related Projects

- [3D Vasculature Reconstruction](#)
- [MonoDepth-vSLAM](#)

---

*For collaboration opportunities or technical inquiries, please [contact me](mailto:rdey@wpi.edu).*
