---
title: pluginlib
date: 2022-04-05 21:10:36
tags:  ROS
categories: ROS
math: true
---

# pluginlib

  In the project, *pluginlib* package provides tools for writing and dynamically loading plugins using the ROS build infrastructure. It's convenient for user to load and using the plugin which is written by other producer. To work, there tools require plugin providers to register their plugins in the package `.xml` of their package.

## 1. Overview

  *pluginlib* is a C++ library for loading and unloading plugins from within a ROS package. Plugins are dynamically loadable classes that are loaded from a runtime library (i.e. shared object, dynamically linked library). With pluginlib, one does not have to explicitly link their application against the library containing the classes -- instead pluginlib can open a library containing exported classes at any point without the application having any prior awareness of the library or the header file containing the class definition. Plugins are useful for extending/modifying application behavior without needing the application source code.

## 2. Example

  In *ros control*, if you would like to develop a self-defined controller, it is the recommended way to using the *pluginlib* and load your controller by *controller_manager*. I will describe how I develop my cartesian controller as following.

## 3. Develop a Plugin

### 3.1 Registering/Exporting a Plugin

  In order to allow a class to be dynamically loaded, it must be marked as an exported class. This is done through the special macro **PLUGINLIB_EXPORT_CLASS**. And it is usinglly put at the end of the file.

```c++
#include <pluginlib/class_list_macros.h>
...
PLUGINLIB_EXPORT_CLASS(cartesian_controller::Cartesian_Velocity_Controller, controller_interface::ControllerBase)
```

### 3.2 The Plugin Description File

  The *plugin description file* is an **XML** file that serves to store all the important information about a plugin in a machine readable format. It contains information about the library the plugin is in, the **name** of the plugin, the **type** of the plugin, etc. My controller description file like this:

```xml
<library path="lib/libcartesian_velocity_controller">
  <class name="cartesian_controller/CartesianVelocityController"            type="cartesian_velocity_controller::Cartesian_Velocity_Controller" 	  base_class_type="controller_interface::ControllerBase">
    <description>
        Cartesian velocity control
    </description>
  </class>
</library>
```

  **Why do we need this file?** We need this file in addition to the code macro to allow the ROS system to automatically discover, load, and reason about plugins. The plugin description file also holds important information, like a description of the plugin, that doesn't fit well in the macro.

### 3.2 Registering Plugin with ROS Package System

  In order for pluginlib to query all available plugins on a system across all ROS packages, each package must explicitly specify the plugins it exports and which package libraries contain those plugins. A plugin provider must point its plugin description file in its `package.xml` inside the *export* tag block.

```xml
 <export>
 	<!-- Other tools can request additional information be placed here -->
  <controller_interface plugin="${prefix}/cartesian_velocity_controller_plugin.xml"/>
 </export>
```

