---
title: SOFA Study Record (Tripod)
date: 2022-07-05 10:07:35
categories: Soft Robot
tags: [Soft Robot, Simulation, Hardware]
math: true
---

## Introduction

This post record how to use the **SOFA** and the **SoftRobots** plugin in order to simulate a soft robot. The first part is dedicated to the set up of the simulation environment and the modeling of a virtual soft robot. Then, this post deals with the control of the real robot associated with the simulation.

Once completed, the knowledge acquired from this post on the robot called "Tripod" can be applied to other soft robots, thanks to the high modularity of SOFA.

Tutorial Requirements:

* You have installed **SOFA** with the **STLIB** and **SoftRobots** plugins.
* You have basic knowledge of the **Python** programming language.
* You have basic knowledge of scene modeling with **SOFA**

This soft robot considered here is actuated by three servomotors connected to a deformable silicone material. Each of them controls the deformation of one 'arm' of the silicone piece. By combining the effects of the three servomotors, it is possible to control the shape of the whole deformable part, which benefits from a theoretically infinite number of degrees of freedom.

Reminder of SOFA's GUI:

Once a scene is loaded in SOFA, you can click on the [Animate] button in the SOFA GUI to start the simulation. Once you have clicked anywhere on the 3D window and with the robot: depending on what was programmed, you can control the simulation to manipulated the robot, or interact with it by pressing `Ctrl+Keys`. Moreover, please note that, in order to send data to the robot, it can be necessary to have administrator rights.

## Details

### STEP 1: Building the Mechanical Model for the soft part & its Visual Model

**At the end of this step, you will be able to:**

* Build the mechanical model of the silicone piece of the robot
* Build the corresponding visual object
* Use meshes for the mechanical description of a soft object

**Reminder of the First Steps tutorial:**

* All the objects simulated in a scene are described in nodes attached to the main node `rootNode`. For this robot, there will be different objects, including one for silicone piece (`ElasticBody`). Each of them is defined with the function `node.createChild()`
* In order to automatically reload the scene when changes made, run the scene with the `-i` option.
* The properties of any object can be accessed by double-clicking on it in the *Graph* panel of the SOFA GUI.
* By right-clicking in the *Graph* panel, the code file can be opened (Open file in editor in the drop-down menu).

The first two steps aim at modeling the deformable silicone piece of the robot. The objects composing it are added one by one with the function `node.createObject()`. A mechanical model of the piece is first created, called `ElasticBody`. As the silicone piece is soft and can be deformed through constraints, it is necessary to describe its entire volume (and not only its center of gravity as it is usually done for rigid objects). Based on its shape, it is discretized: here it is decomposed into tetrahedral elements, linking all the points - or nodes - of the volume. This discretization produces the following types of elements: tetrahedrons, triangles, lines, and points. These elements are listed into a `MechanicalObject` named `"dofs"`. The mesh file describing the discretization into tetrahedral element is `"tropod_mid.stl"`. The data provided by the meshes must be organized and stored into different data fields (one for the positions of the nodes, one for the triangular elements, ...) before SOFA uses them for computation. This formatting is done by means of code blocks called loaders. The loader used here, `MeshSTLLoader` is designed for `STL` files.

The mass distribution of the piece is implemented, as well as time integration scheme (implicit Euler method) and a solver (here the quick resolution method using the `LDL` matrix decomposition).

Then, a Visual model of the piece is built, using the same mesh file as for the mechanical model. Because it was decided (for the sake of simplicity) to use the same meshing for both models, the loader is introduced in the rootNode.

Finally, the two representations are linked together by a mapping, which allows to adapt the visual model to the deformations of the mechanical model of the soft piece. As the same mesh is used for both the mechanical  and the visual model, the mapping is simply mirroring the mechanical model to adapt the visual one. This is why we use an `IdentityMapping` here.

```python
import Sofa
from stlib3.scene import Scene

def createScene(rootNode):
    # Setting the parameters: Gravity, Assuming the length unit is in millimeters.
	scene = Scene(rootNode, gravity=[0.0, -9810, 0.0], plugins=['SofaparseSolver', 'SofaOpenglVisual'], iterative=False)
	scene.addMainHeader()
	scene.addObject('DefaultAnimationLoop')
	scene.addObject('DefaultVisualmanagerLoop')
	
    # Setting the parameters: Timestep in seconds
	rootNode.dt = 0.01
	
    # Graphic Modeling of the legends associated to the servomotors
	blueprint = Sofa.Core.Node('Blueprints')
    blueprint.addObject('MeshSTLLoader', name='loader', filename='data/mesh/blueprint.stl')
    blueprint.addObject('OglModel', src='@loader')
    scene.Settings.addChild(blueprint)
    
    # Tool to load the mesh file of the silicone piece. It will be used for both the mechanical and the visual models.
    scene.Modelling.addObject('MeshSTLLoader', name='loader', filename='data/mesh/tripod_mid.stl')
    
    # Basic mechanical modeling of the silicone piece
    elasticbody = scene.Modelling.addChild('MechanicalModel')
    elasticbody.addObject('MechanicalObject', name='dofs', 
                          position=scene.Modelling.loader.position.getLinkPath(),
                          showObject=True, showObjectScale=5.0,
                          rotation=[90.0, 0.0, 0.0])
    elasticbody.addObject('UniformMass')
    
    # Visual object
    visual = scene.Modelling.addChild('VisualModel')
    # The mesh used for the Visual object is the same as the one for the MechanicalObject,
    #and has been introduced in the rootNode
    visual.addObject('OglModel', name='renderer',
                     	src=scene.Modelling.loader.getLinkPath(),
                    	color=[1.0, 1.0, 1.0, 0.5])
    
    # A mapping applies the deformations computed on the mechanical model (the input parameter)
    # to the visual model (the output parameter)
    visual.addObject('IdentityMapping',
                    	input=elasticbody.dofs.getLinkPath()
                    	output=visual.renderer.getLinkPath())
    scene.Simulation.addChild(scene.Modelling)
```

#### Exploring the scene

* Try to orient the object differently on the 3D window by modifying its properties. Step 1 of the tutorial can be used as a guideline.
* In the *View* panel of SOFA GUI, by enabling the *Options*, you can see the discretization of the silicone piece into tetrahedral elements.
* Identify the white squares, each representing one point (one degree of freedom) of the `MechanicalObject`, on which the deformations are computed.

#### Remarks

* SOFA implements default length and time units, as well as a default gravity force. The user defines his own time and space scale by defining the constants of the model. Here, the gravity of our simulation is defined such as the length unit is in centimeters; and the time unit chosen is the second, which means that the time step is of one millisecond.
* There is a graphic modeling in the scene to display the legends associated with the servomotors, that are described in the file `blueprint.stl`.

<img src="/image/SOFA/Tripod/1.png" style="zoom:80%;" />



### STEP 2: Modeling the possible deformations of the soft material

**At the end of this step, you will be able to:**

* Build an elastic deformation model based on the Finite Element Method
* Understand what a `ForceField`is

Unlike the rigid objects modeled in the First Steps tutorials, the silicone piece is deformable, and as such, requires additional components describing the behavior of the material when it is submitted to mechanical constraints.

In order to implement the deformation behavior, the `MechanicalObject` must be connected to one or multiple `ForceFileds`. These `ForceFields` are in charge of computing the internal forces that will guide the deformation of the soft piece. Many different mechanical behaviors exist and it is important to understand the one that best approximates the behavior of the real object. In particular, it is important to know how soft or stiff the material is, as well as if it has an elastic behavior or a more complex one (hyperelastic, plastic, etc...). In our case, it has been chosen to implement a law of elastic deformation, modeled using the Finite Element Method(`FEM`). Its parameters are the Young modulus, and the Poisson ratio.

```python
elasticbody.createObject("TetrahedronFEMForceField", youngModulus=250, poissonRation=0.45)
```

 In SOFA, the `ElasticMaterialObject` from `stlib3.physics.deformable` provides a ready to use prefabricated object to easily add such an object in our scene. It defines the whole mechanical model of a deformable elastic object.

```python
ElasticMaterialObject.createPrefab(node, volumeMeshFileName, name, rotation, traslation, surfaceColor, poissonRatio, youngMoudulus, totalMass, solver)
```

However, before using this prefabricated object, let's first build our own, based on a corotational Finite Element Method with a tetrahedral volumetric representation of the shape. The mesh `tripod_mid.gidmsh` used to store the shape of the deformable object was built with the GiD mesh generator. Starting from the model obtained in the last step, the parameters of the elastic deformation are added to the silicone piece with a `ForceField` component.

```python
import Sofa
from stlib3.scene import Scene

def createScene(rootNode):
    scene = Scene(rootNode, gravity=[0.0, -9810, 0.0],
                  plugins=['SofaSparseSolver', 'SofaOpenglVisual', 'SofaSimpleFem', 'SofaGraphComponent'], iterative=False)
    scene.addMainHeader()
    scene.addObject('DefaultAnimationLoop')
    scene.addObject('DefaultVisualManagerLoop')
    scene.dt = 0.01
        
    # It is possible to visualize the 'forcefields' by doing
    scene.VisualStyle.displayFlags = 'showForceFields'
    
    # Change the stiffness of the spring while interacting with the simulation
    scene.Settings.mouseButton.stiffness = 1.0
    
    # Graphic modelling of the legends associated to the servomotors
    blueprint = Sofa.Core.Node("Blueprints")
    blueprint.addObject('MeshSTLLoader', name='loader', filename='data/mesh/blueprint.stl')
    blueprint.addObject('OglModel', src='@loader')
    scene.Modelling.addChild(blueprint)

    # To simulate an elastic object, we need:
    # - a deformation law (here linear elasticity)
    # - a solving method (here FEM)
    # - as we are using FEM we need a space discretization (here tetrahedron)
    elasticbody = scene.Modelling.addChild('ElasticBody')

    # Specific loader for the mechanical model
    elasticbody.addObject('GIDMeshLoader',
                          name='loader',
                          filename='data/mesh/tripod_low.gidmsh')
    elasticbody.addObject('TetrahedronSetTopologyContainer',
                          src='@loader',
                          name='tetras')
                          
    mechanicalmodel = elasticbody.addChild("MechanicalModel")                      
    mechanicalmodel.addObject('MechanicalObject',
                          name='dofs',
                          position=elasticbody.loader.position.getLinkPath(),
                          rotation=[90.0, 0.0, 0.0],
                          showObject=True,
                          showObjectScale=5.0)
    mechanicalmodel.addObject('UniformMass', 
                          name="mass", 
                          totalMass=0.032)

    # ForceField components
    mechanicalmodel.addObject('TetrahedronFEMForceField',
                          name="linearElasticBehavior",
                          youngModulus=250,
                          poissonRatio=0.45)
    
    # Visual model
    visual = Sofa.Core.Node("VisualModel")
    # Specific loader for the visual model
    visual.addObject('MeshSTLLoader',
                     name='loader',
                     filename='data/mesh/tripod_mid.stl',
                     rotation=[90, 0, 0])
    visual.addObject('OglModel',
                     src=visual.loader.getLinkPath(),
                     name='renderer',
                     color=[1.0, 1.0, 1.0, 0.5])
    scene.Modelling.ElasticBody.addChild(visual)

    visual.addObject('BarycentricMapping',
                     input=mechanicalmodel.dofs.getLinkPath(),
                     output=visual.renderer.getLinkPath())
   
    scene.Simulation.addChild(elasticbody)                 
```

