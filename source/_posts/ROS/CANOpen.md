---
layout: ros
title: CANOpen
date: 2022-04-05 21:59:08
tags: hardware roscanopen
categories: ROS
math: true
---
# ros_canopen

## 1. Overview

  This packages provides support for *CANopen* devices within ROS. It can be divided into different parts:

* *CAN* layer abstraction
* *CANopen* master with device/object management
* Profile-specific support, currently only for 402 profile (drives and motion control)
* ROS interfaces, motion control through *ros_control*
<!-- {% asset_img 1.png This is an example image %} -->

<img src="/image/1.png" style="zoom:80%;" />

## 2. *socketcan_interface*

### 1) Overview

  This packages provides a generic CAN interface class and a SocketCAN-based driver implementation. The interface is divided into three parts:

* *StateInterface*: Listener interface for the state of the driver

* *CommInterface*: Listener interface for receiving messages and send functionality

* *DriverInterface*: inherits from both and adds management interfaces

  The listeners are based on `std::function` (since melodic) and use RAII-pointers. The SocketCAN driver is based on *Boost.Asio* and provides concurrent access to SocketCAN interfaces. It is the default CAN implementation used throughout ros_canopen and requires linux kerner 2.6.25 or newer.

### 2) Tested Drivers and Devices

  The SocketCAN driver interface should work with all SocketCAN network interfaces. The linux mainline kernel already includes some device drivers, other devices are supported by vendor-provided implementations.

### 3) Preparation

#### 3.1 Load driver

  Set-up udev rules or type manually:

```bash
$ sudo modprobe peak_usb # kernel driver, since 3.11
```

  `modprobe` is a linux program originally written by Rusty Russell and used to add a loadable kernel module to the linux kernel or to remove a loadable kernel module form the kernel.

#### 3.2 Initialize NIC (Network Interface Card)

  Bring-up CAN NIC (Linux kernel drivers):

```bash
$ sudo ip link set can0 up type can bitrate 500000 # adjust bitrate as needed
```

  Further options are available, please consult SocketCAN Readme, section 6.5.

### 4) Tools and Debugging

#### 4.1 socketcan_dump

  Simple test program provided with the package, just prints the received messages:

```bash
$ rosrun socetcan_interface socketcan_dump can0
```

#### 4.2 Netlink status

```bash
$ ip -details -statistics link show can0
```

#### 4.3 can-utils

Feature-rich tool suite for SocketCAN, install with:

```bash
$ sudo apt-get install can-utils
```

* candump
* cansend
* canbusload

## 3. *socketcan_bridge*

### 1) Overview

  The packages provides functionality to expose CAN frames form *SocketCAN* to a ROS topic. Internally it uses the *socketcan_interface* form the *ros_canopen* package, as such it is capable of dealing with both normal and extended CAN frames.

## 4. *canopen_master*

### 1) Overview

  This packages contains a master implementation of the CANopen DS 301 protocol. It provides libraries to connect to CANopen devices and access their data objects via high-level interfaces.

### 2) Features

  The implementation provides support for many CANopen interfaces and services:

* High-level object dictionary for all simple types, except booleans (1bit)
* EDS/DCF parser
* Automatic configuration based on communication objects
* Full SDO client implementation
* PDO subscriber/publisher for different transmission types
* EMCY handling with diagnostics interfaces
* NMT with heartbeat (client), without node-guarding
* Plug-in system for bus synchronization and the SYNC objects

### 3) *CANopen* basics

#### 3.1 NodeID

  Each device is assigned a node ID in range between 1 and 127. It determines the CAN frame the device and listens to, so it must be unique (perbus).

#### 3.2 Object dictionary

  All data is stored as objects, which are addressed with 4-digit hexadecimal numbers and an optional hexadecimal 8-bit sub index. The object types range from simple integer values to complex structs and arrays.

  The object dictionary contains all objects (data & types) and is structured based on the index, notable ranges are:

* 1000 - 1FFF: Communication objects
* 2000 - 5FFF: Manufacturer-specific objects
* 6000 - 6FFF: Profile-specific objects (for first logical device)

#### 3.3 Network management (NMT)

  The network management includes a state machine for the overall status (pre-operational, operation, stopped) of the devices and service to control the states. Further it can be used to monitor the devices.

#### 3.4 Service data objects (SDOs)

  Client/Server interface for reading and writing objects. SDO cannot be used with stopped devices.

#### 3.5 Process data objects (PDOs)

  Publisher/Subscriber interface for reading and writing objects. Limited to 8 bytes per PDO, can contain multiple (simple) objects. Not all objects can be mapped to PDOs, this depends on the object and the device. PDOs are available in the operational state only.

#### 3.6 Synchronization object (SYNC)

  Periodically object sent by the master to synchronize the devices of one bus. PDOs might be set-up to be synchronous (recommended for *ros_canopen*).

#### 3.7 Emergency objects (EMCY)

  The nodes send these objects on critical errors. The standard lists a wide range of errors code, profile can have additional codes.

### 4) EDS/DCF files

  The electronic data sheet and the device configuration files are standardized in CiA 306 DS. These are INI files that contain all objects descriptions. The new XML based XDD is currently not supported.

  The EDS just contains the defaults, the DCF can include specific parameter values, however *canopen_master* treats both alike. All entries that have parameter values different from the defaults are written to the device at initialization. PDO objects have additional restrictions and are handled separately.

  All manufactures should provide EDS or DCF files. The editor Vector CANeds 3.6 is available freely and plays well with wine. The 3.7 version is free as well, but bundled to the demos version of the big CAN software suites. However, it can extract the object dictionary from any CANopen device.

### 5) Master plug-ins

  Each CANopen bus shall have up to one master that is responsible for the synchronization. The driver has preliminary support for inter-process communication which should allow to run two driver instances on one bus. This implementation is currently not ready for production use however.

## *canopen_chain_node*

### 1) Overview

  This packages contains the ROS interface for *socketcan_interface* and *canopen_master*. It can be used as a stand-alone ROS node, but as well as a base class for profile specific ROS interfaces, e.g. *canopen_motor_node*.

  It manages a CANopen bus with one or more nodes, which are configures with CANopen EDS/DCF files. The ROS node runs a control loop with CANopen SYNC interval (or with an alternative update interval if the SYNC protocol is not used).

### 2) Configuration

  The configuration can be split into different parts. First the bus has to be set-up, these settings are shared for all node. Then all must be configured with their CANopen and ROS interfaces. All parameters must be loaded into the nodes private namespace.

