---
title: micro_ros
date: 2022-12-21 23:13:51
tags: [ROS, STM32]
---

# Micro-ROS on STM32 micro-controller

  This is the unofficial tutorial for setting up the micro-ROS on STM32 micro-controller. I have successfully set up the micro-ROS on the `STM32F446RET6` and tested with `Ubuntu20.04`. Following this tutorial, you could set up it on your own micro-controller, but for the arduino , you might refer some other references. In my opinion, there are the same one and it could be easily to find out the official document on the Github.

## Introduction

### Features

  The Micro-ROS offers seen key features that make it ready for use in your micro-controller-based robotic project:

* Micocontroller-optimized client API supporting all major ROS concepts: Micro-ROS brings all major core concepts such as nodes, publish/subscribe, client/service, node graph, life-cycle, etc. onto microcontrollers (MCU). The client API of Micro-ROS is based on the standard ROS 2 Client Support Library (`rcl`) and a set of extensions and convenience functions;
* Seamless integration with ROS 2;
* Extremely resource-constrained but flexible middle ware;
* Multi-RTOS support with generic build system;
* Permissive license;
* Vibrant community and ecosystem;
* Long-term maintainability and interoperability;

### Architecture

![img](https://micro.ros.org/img/micro-ROS_architecture.png)

  Micro-ROS follows the ROS 2 architecture and makes use of its middleware pluggability to use `DDS-XRCE`, which is optimized for microcontrollers. Moreoer, it uses POSIX-based RTOS(FreeRTOS, Zephyr, or NuttX) instead of Linux. Dark blue components are developed specially for Micro-ROS. Light blue components are taken from the standard ROS 2 stack.

### Requirements

* STM32 micro-controller (ex: Necleo STM32F446R6)

* Linux PC with installed ROS 2
* STM32CubeMX
* STM32CubeIDE

## Set up for Micro-controller

### STMCubeMX: Generate Basic Code

  Using STMCubeMX, we can initialize the pin with in the GUI. Specially, I setup the `FreeRTOS`, `UART4`, `TIMER1`, `GPIO`and basic pin initialized autonomously. Among them, the `FreeRTOS` is one of the real time control architecture, and I will use it to control the device in the real time mode. The `UART4` is the connection protocol to the PC and it is the very high priority to interrupt. I use two channel of `TIMER1` to generate `PWM` signal and control device. Then the `GPIO` is used to control the input signal of the switch circuit fabricated by transistor.

![image-20221220144432706](/home/brl/.config/Typora/typora-user-images/image-20221220144432706.png)

  Then choose the generate mode with `Makefile` and do it.

### `Makefile`

  we need modify the generated `Makefile` to include the following code before the `build` operation:

```makefile
#######################################
# micro-ROS addons
#######################################
LDFLAGS += micro_ros_stm32cubemx_utils/microros_static_library/libmicroros/libmicroros.a
C_INCLUDES += -Imicro_ros_stm32cubemx_utils/microros_static_library/libmicroros/microros_include

# Add micro-ROS utils
C_SOURCES += micro_ros_stm32cubemx_utils/extra_sources/custom_memory_manager.c
C_SOURCES += micro_ros_stm32cubemx_utils/extra_sources/microros_allocators.c
C_SOURCES += micro_ros_stm32cubemx_utils/extra_sources/microros_time.c

# Set here the custom transport implementation
C_SOURCES += micro_ros_stm32cubemx_utils/extra_sources/microros_transports/dma_transport.c

print_cflags:
	@echo $(CFLAGS)
```

### micro_ros_stm32cubemx_utils

  This step, we need to clone the repository into the firmware location. And this repository is aims to ease the micro-ROS integration in a `STM32CubeMX/IDE` project.

**Note: These commands must be run under the Firmware folder**

```bash
git clone https://github.com/micro-ROS/micro_ros_stm32cubemx_utils.git
cd micro_ros_stm32cubemx_utils
git checkout foxy
```

Then using the docker, to execute the static library generation tool. Compiler flags will retrieved automatically from our `Makefile` and user will be prompted to check if they are correct.

**Note: These commands must be run under the Firmware folder**

```bash
sudo docker pull microros/micro_ros_static_library_builder:foxy
sudo docker run -it --rm -v $(pwd):/project --env MICROROS_LIBRARY_FOLDER=micro_ros_stm32cubemx_utils/microros_static_library microros/micro_ros_static_library_builder:foxy
```

After the installing process, you can find out the following output which means installed successfully.

```bash
Summary: 63 packages finished [53.7s]
  40 packages had stderr output: action_msgs actionlib_msgs builtin_interfaces composition_interfaces control_msgs diagnostic_msgs example_interfaces geometry_msgs libyaml_vendor lifecycle_msgs micro_ros_msgs microxrcedds_client nav_msgs rcl rcl_action rcl_interfaces rcl_lifecycle rcl_logging_noop rclc rclc_lifecycle rclc_parameter rcutils rmw rmw_implementation rmw_microxrcedds rosgraph_msgs rosidl_runtime_c rosidl_typesupport_c rosidl_typesupport_microxrcedds_c sensor_msgs shape_msgs statistics_msgs std_msgs std_srvs stereo_msgs test_msgs tf2_msgs trajectory_msgs unique_identifier_msgs visualization_msgs
```

### micro_ros_setup

