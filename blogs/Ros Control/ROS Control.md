# ROS Control

  The `row_control` packages takes as input the joint state data from your robot's actuator's encoders and an input set point. It uses a generic control loop feedback mechanism, typically a PID controller, to control the output, typically effort, sent to your actuators.

<img src="/Users/mieongsun/Downloads/smbarber/blogs/Ros Control/1.png" alt="1" style="zoom:77%;" />

<center>Fig 1. Diagram source in ros_control/documentation</center>

  `ros control` helps you writing controllers including real-time constraints, transmissions and joint limits. It provides tools for modeling the robot in software, represented by the robot hardware interface (`hardware_interface::RobotHW`), and comes with ready to use out of the box controllers， thanks to a common interface (`controller_interface::ControllerBase`) for different applications which then talk to third-party tools. The **important aspects** to highlight is that the hardware access to the robot is **decoupled**, so you don't expose how you talk to your robot. This means that controllers and your robot hardware structure are decoupled so you can develop them **independently** and in **isolation** which enables you to share and mix and match controllers.

  When you want to write your robot backend, there's a possibility to leverage an out of the box simulation backend (`gazebo_ros_control` plugin). After you've tested your robot in simulation, and want to work on the real robot hardware, you will want to write your custom hardware backend for your robot.

  There is already a set of existing controllers (`ros_controllers` meta package) that are robot agnostic, which should hopefully enable you to leverage some of those. However, if your application requires it, then there are tools (`controller_interface`) that help you also implement custom controllers. `ros control` also provides the `controller manager` for controlling the life cycle of your controllers.

  Another **important aspects** is that `ros control` is real-time ready in the sense that if your application has real-time constraints then you can use `ros control` with it.

  In summary, the goals of the `ros control` are to

* lower the entry barrier for exposing hardware to ROS.
* promote the reuse of control code in the same spirit that ROS has done for non-control code, at least non-real-time control code.
* provide a set of ready-to-use tools.
* have a real-time ready implementation.

