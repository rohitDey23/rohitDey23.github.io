---
title: Navigating Autonomous Ground Vehicle in Gazebo with ROS2
date: 2024-05-10 10:20:06 -5000
categories: [Robotics, Navigation,  Autonomous Driving]
tags: [ros2, gazebo, robotics, agv, robotics, navigation, autonomous_driving] 
---
## >> Objectives: 
<p align='justify'>This post documents my progress in developing and testing an Autonomous Ground Vehicle (AGV) navigation system in a simulated environment using <a href='https://gazebosim.org/home'>Gazebo</a> and <a href='https://docs.ros.org/en/humble/index.html'>ROS2</a>. The project would be sub-divided into following sub-task to make it more managegable and structured:</p>

- [x] **Task 1** : Creating three Gazebo world versions for testing our naviagtion stack, which would include: 
1. a) Multiple target locations with scattered objects that are not obstacles(used for - landmarks mainly).
2. b) Multiple target locations with scattered static obstacles and landmarks.
3. c) Multiple target locations with dynamic obstacles and static landmarks.

- [x] **Task 2** : Modeling our AGV and importing it into the gazebo world ash URDF files
- [x] **Task 3** : Controlling the AGV using  teleop controller.
- [x] **Task 4** : Implementing  different navigation algorithms (A*, DWA, RRT & RRT*) on the different world scenarios.
- [x] **Task 5** : Compare the above algorithms with [Nav2](https://docs.nav2.org/about/index.html#id1) stack by Open Navigations


## >> Task 1: Creating Gazebo Worlds