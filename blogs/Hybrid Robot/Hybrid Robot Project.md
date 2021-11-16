# Hybrid Robot Project

**Platform Producer:** *Beijing Union University (BUU)*

Completed by *Mingshan He* (First Author & Engineer)



## Introduction

  This project are mainly about the hybrid robot which contains the moving platform (sage) and collaborate robot (xMate). The moving platform are carrying the lidar and the camera (Realsense2 Di145). On the other hand, the collaborate robot are carrying the gripper (PGI140) and the camera (Realsense2 Di145). Hence, it can complete the tasts which need the movement and grasp. The destination in this project is that the whole platform moving in the known environment with SLAM and grasp a object directed by the operator.

## Constituent

### 1. Computer Vision

  Before this project, I have prepared the knowledge about the camera calibration and running the calibration programm using the hand-eye camera. In this project, it need to recognize the object to grasp by the camera on the collaborate arm and the place to move by the one on the moving platform. 

### 2. SLAM

  The navigation module are refered the framework of the turtlebot. Map creating by the *gmapping* and navigation using *graph*.

### 3. Control Framework

  Mainly for the robot control.

## Details

  The `*`  flag means the file is created and edited by Mingshan He.

### 1. Computer Vision

  In this module, it need to launch the camera to recognize. This project has been designed to recognize the position and orientation of the object by using the *aruco* tag. In the *aruco_ros* folder contains the file to launch the recognition algorithm and it will publish the result in the particular topic as pose message. There is a gripper (DH Robotics PGI140) in the end of robot arm which supply the producer the gripper function. The grasp function is developed in Cartesian coordinate, which has a better performance.

#### 1.1 Realsense2 Camera

  This folder are containing the libraries and drivers for the Intel Realsense2 camera. You can use the following order to drive the camera and view the image from it.

```bash
$ roslaunch realsense2_camera rs_rgbd.launch
```

  After launch this file, the node of camera has been registered in the ros master. If you wanna to have a look at the image callback from the camera, you can use the `rqt` plugins.

#### 1.2 Calibration

  **Why Calibration?** First, the gripper need to arrive at the appointed position which solved from the image. During the camera capture the image, there is a distance between the camera and gripper. And it is exactly influence the correctness of grasp. Hence, the calibration is need to compensate it. **What's Calibration?** There are several approaches in the calibration. In this project, I have used the *aruco* tag to calibrate. After calibration, the programm will provide a offset contains the distance of linear and angular. The frame of camera can be modeled by this method and the transform between the end of effector and camera is defined. 

#### 1.3 TF Transform

  **TF** is the shorten of the transform. It is the prevalent packages to presant the transform of the joints in ros. The link and joints of robot can be described in *robot_description* as urdf file. If the externed device like camera or gripper is installed at the end of robot arm, you can use the *tf* package to broadcaste the transform between the device and the end instead of modifing the urdf file frequently. 

  In this project, the devices of gripper and camera are applied to accomplish the functions. As all we know, the gripper and joint7 has the same axis which means there are only distance along the revolute axis of joint7. And the distance can be compensated by the camera, so I just create the transform between the camera and link7 which is the end of the robot. You can use the following order to launch the tf service to broadcast the camera frame based on the link7.

```bash
$ roslaunch realsense2_camera camera.launch
```

  For the recoginition of object, I have used the *aruco* tag which have mentioned on the bellow. So it also need to use the result of the recognition to broadcast the transform between the camera and the tag. The order is described as follows.

```bash
$ rosrun realsense2_camera aruco_tf_node
```

  In this node, it has also published the information of tag in the topic. The robot controller need it to arrive at the appointed position to grasp object.

### 2. SLAM(Simultaneous Localization and Mapping)

  Navigation is the ability of a mobile robot to determine its position in the environment where it is located (localization), and to plan and execute the path to a target location. It enables autonomous avoiding both static and dynamic obstacles. For navigation to work, it is very important to have a map of the environment.

#### 2.1 Algorithm Overview

  **AMCL** and **Gmapping** (or some other SLAM algorithm) are important parts for mapping process. AMCL (Adaptive Monte Carlo Localization) is a probalistic localization system for a robot moving in 2D. Gmapping is an implementation of a specific SLAM algorithm. And it is an algorithm that is already implemented in ROS.

#### 2.2 Gmapping: Create Map

  The *Gmapping* package contains the ROS node named *slam_gmapping*, which builds a 2D map, using data on laser sensor measurements and the data on the position of the mobile robot. The node *slam_gmapping* (in ros this node is named *gmapping*) is subscribed to the topics `/scan` and `/tf`, in order to obtain the necessary data to build the map. This node reads data from the laser and transformations and then creates **OGM** (Occupancy Grid Map) based on that data. The map that is being created is being published on the topic `/map` throughout the mapping process. Topic `/map` uses a message of type `nav_msgs/OccupancyGrid`. In **OGM**, the occupancy is represented by the integer value in the range from 0 to 100. A value of 0 means completely unoccupied and 100 means fully occupied (e.g., wall), and a special value is -1, which means an unknown are.

```bash
$ roslaunch four_wheels scan_mode.launch
```

##### 2.2.1 More details

  ***Problem Formulation:*** **Given** observations $z_{l:t} = z_l,...,z_t$, and odometry measurements $u_{2:t} = u_l,...,u_t$; **Find** Posterior $p (x_{l:t}, m | z_{l:t}, u_{2:t})$ With m a grid map

  ***Key Ideas:*** 

  **Rao-Blackwellized Particle Filter**: Each particle equals the sample of history of robot poses and posterior over maps given the sample pose history; approximate posterior over maps by distribution with all probability mass on the most likely map whenever posterior is needed.

  **Proposal distribution $\pi$:** Approximate the optimal sequential proposal distribution $p^{*}(x_t) = p(x_t | x^i_{l:t-l}, z_{l:t}, u_{l:t}) \propto p(z_t | m^i_{t-l}, x_t) p(x_t | x^i_{t-l}, u_t)$ . First, find the local optimum argmax $p^*(x)$; Second, sample $x^k$ around the local optimum, with weights $w^k = p^{*}(x^k)$; Third, fit a Gaussian over the weighted samples; Finally, this Guassian is an approximation of the optimal sequential proposal $p^*$.

  **Weight** update for optimal sequential proposal is $p(z_t | x^i_{l:t-l}, z_{l:t-l}, u_{l:t}) = p(z_t | m^i_{t-l}, x^i_{t-l}, u_{t-l})$, which is efficiently approximated from the same samples as aboved

  **Resampling ** based on the effective sample size $S_{eff}$

***Algorithm:*** **Improved RBPF for Map Learning**

**Require**:

  $S_{t-1}$, the sample set of the previous time step

  $z_t$, the most recent laser scan

  $u_{t-1}$, the most recent odometry measurement

**Ensure:**

  $S_t$, the new sample set

  $S_t$ = {}

  **for all** $s^{(i)}_{t-1} \in S_{t-1}$ **do**

​    ...



Referred from UC. Berkerly

#### 2.3 AMCL

  ACML is a probabilistic localization system for a robot moving in 2D. It implements the adaptive (or KLD-sampling) Monte Carlo localization approach (as described by Dieter Fox), which uses a particle filter to track the pose of a robot against a know map.

#### 2.2 Initialization

  To initial the robot pose, which can not localization by theirself.

```bash
$ rosrun move_control init_pose
```

  To set the goal of the destination, which is supposed to be known.

```bash
$ rosrun move_control set_goal
```



#### 2.3 Navigation

```bash
$ roslaunch four_wheels navigation_mode.launch
```





### 3. Robot Arm (ROKAE xMate 3 Pro & DH Robotics PGI140)

#### 3.1 Control Device

  The hardware of control device has been used the industrial control computer which produced by Bing Han. In the programm, it has contained all of the algorithm of the robot and hardware device. The code has not been opensourced, so I have not any details to describe. For control the robot by this, you need to login the computer in ssh and run the executable files in following command.

```bash
$ ssh root@192.168.188.99 (password: root)
$ ./DeviceDriverMain --type 4 --cycle 1000
```

```bash
$ ssh root@192.168.188.99 (password: root)
$ ./robot/HYYRobotMain
```

#### 3.2 ROS Control

  The ros control is the prevalent control package in ros, I have concluded it in the other blog. If you have interest in it, you can click the link. You can run the ros control as following command and start the ros controller.

```bash
$ ssh root@192.168.188.99 (password: root)
$ roslaunch device_driver device_driver.launch
```

```bash
$ ssh root@192.168.188.99 (password: root)
$ roslaunch device_driver start_controller.launch controller_name:=hyy_trajectory_controller
```

  If you wanna to stop the controller or power off the robot. You can use the following command.

```bash
$ ssh root@192.168.188.99 (password: root)
$ roslaunch device_driver stop_controller.launch controller_name:=hyy_trajectory_controller
```

#### 3.3 KDL(Kinematic & Dinamic Library)

  In this project, I have developed the moving library in Cartesian coordinate. And the device controller also has the *MoveL* function which has the same function.

```bash
$ roslaunch move_control cartesian_demo_node
```



#### 3.4 Gripper (DH Robotics PGI140)

*  Serial and Modbus 485

```bash
$ rosrun move_control Gripper_node
```

