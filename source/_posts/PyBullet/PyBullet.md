---
title: PyBullet
date: 2022-10-14 22:12:51
tags: [PyBullet, Simulation]
math: true
---
# PyBullet

## Introduction

  PyBullet is a fast and easy to use Python module for robotics simulation and machine learning, with a focus on sim-to-real transfer. PyBullet provides forward dynamics simulation, inverse dynamics computation, forward and inverse kinematics, collision detection and ray intersection queries.

  PyBullet can be easily used with `TensorFlow` and `OpenAI Gym`. Researchers from Google Brain, Stanford AI Lab, INRIA and many other labs use PyBullet. If you use PyBullet in your research, please add a citation.

### Hello PyBullet World 

Here is a PyBullet script.(`Hello_PyBullet_World.py`)

```python
import pybullet as p
import time
import pybullet_data

physicsClient = p.connect(p.GUI)#or p.DIRECT for non-graphical version

p.setAdditionalSearchPath(pybullet_data.getDataPath()) #optionally

p.setGravity(0,0,-10) # Gravity

planeId = p.loadURDF("plane.urdf") # load basic plane URDF

startPos = [0,0,2] # the Original Pos of the object 

startOrientation = p.getQuaternionFromEuler([0,0,0]) # the Original Ori with the Euler of the object

boxId = p.loadURDF("r2d2.urdf",startPos, startOrientation) # load object URDF

# Set the simulation time and time sleep of the step
# I think the step time makes the step time discrete and makes the calculate resourses more efficency.
# You could modify this value to accelerate the simulation or deaccelerate.
for i in range (10000):
    p.stepSimulation()
    time.sleep(1./240.)
cubePos, cubeOrn = p.getBasePositionAndOrientation(boxId)
print("[USER INFO]:")
print(cubePos,cubeOrn)
p.disconnect()

```

#### Functions

##### connect, disconnect, bullet_client

  After importing the PyBullet module, the first thing to do is **connecting** to the physics simulation. PyBullet is designed around a client-server driven API, with a client sending commands and a physics server returning the status.

  You can provide your own data files, or you can use the `PyBullet_data` package that ships with PyBullet. For this, import `pybullet_data` and register the directory using `pybullet.setAdditionalSearchPath(pybullet_data.getDatapath())`. For other connection types:

```python
pybullet.connect(pybullet.DIRECT)
pybullet.connect(pybullet.GUI, options="--opengl2")
pybullet.connect(pybullet.SHARED_MEMORY,1234)
pybullet.connect(pybullet.UDP,"192.168.0.1")
pybullet.connect(pybullet.UDP,"localhost", 1234)
pybullet.connect(pybullet.TCP,"localhost", 6667)
```

  You can **disconnect** from a physics server, using the physics client Id returned by the connect call (if non-negative). A `DIRECT` or `GUI` physics server will shutdown A separate (out-of-process) physics server will keep on running.

  If you want to use multiple independent simulations in parallel, you can use `pybullet_utils.bullet_client`.

##### setGravity

  By default, there is no gravitational force enabled. `setGravity` lets you set the default gravity force for all objects.

##### loadURDF, loadSDF, loadMJCF

  The `loadURDF` will send a command to the physics server to load a physics model from a Universal Robot Description File. There are some loadURDF arguments:

  `basePosition` create the base of the object at the specified position in world space coordinates; `baseOrientation` create the base of the object at the specified orientation as world space quaternion [X,Y,Z,W]; `useMaxmalCoordinates`by default, the joints in the URDF file are created using the reduced coordinate method; `useFixedBase` force the base of the loaded object to the static; `globalScaling` will apply a scale factor to the URDF model.

  The `loadSDF` command only extracts some essential parts of the SDF related to the robot models and geometry, and ignores many elements related to cameras, lights and so on The `loadMJCF` command performs basic import of MuJoCo MJCF xml files.

##### saveState, saveBullet, restoreState, removeState, saveWorld

  When you need deterministic simulation after restoring to a previously saved state, all important state information, including contact points, need to be stored.(`SaveData.py`)

##### createCollisionShape, VisualShape

  Although the recommended and easiest way to create stuff in the world is using the loading functions (loadURDF/SDF/MJCF/Bullet), you can also create collision and visual shapes programmatically and use them to create a multi body using createMultiBody. You can create a visual shape in a similar way to creating a collision shape, with some additional arguments to control the visual appearance, such as diffuse and specular color. (`createMultiBodyLinks.py` and `createVisualShape.py`)

##### stepSimulation, performCollisionDetection

  stepSimulation will perform all the actions in a single forward dynamics simulation step such as collision detection, constraint solving and integration. The default time step is **1/240** second, it can be changed using the `setTimeStep` or `setphysicsEngineParameter` API. The performCollisionDetection API will perform just the collision detection stage of the stepSimulation.

## Controlling a robot