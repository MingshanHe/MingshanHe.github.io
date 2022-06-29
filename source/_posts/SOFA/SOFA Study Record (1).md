---
title: SOFA Study Record (1) 
date: 2022-06-28 10:20:00
categories: Soft Robot
tags: [Soft Robot, Simulation]
---

## Introduction

The idea of creating an open-source platform for physics-based simulation first arouse in the rooms of the CIMIT in Boston in 2000. It took about four years before a white paper  describing the concept of the architecture of SOFA was written in 2004. For the first time, this white paper foresaw a collaborative project between the CIMIT and Inria. Soon, further Inria teams joined the project. Early in 2005, the first live demonstration of SOFA was performed at the occasion of MMVR in Los Angeles. Subsequently, the American Department of Defense decided to fund the development of SOFA while Inria actively supported the project with the first SOFA engineers.

Today, SOFA gathers about 15 years of research in physics simulation. Many publications were accepted, several simulators were developed and five startups were created. The research topics were diverse:

* Solid mechanics with the simulation of the brain, the ear, the bones, the heart, the liver,
* Fluid dynamics with the simulation of fat filling and blood flow in aneurysms,
* Thermodynamics with thermo-ablation of tumors,
* and many other topics as image processing, animation or biological applications!

## Getting Started

I have built the SOFA framework with the [official documentation](https://www.sofa-framework.org/community/doc/getting-started/build/linux/). I have no additional idea in the build process, and I think the documentation is detailed enough to start it. Hope all of you could build it successfully.

## Using SOFA

### run SOFA

The default compilation of SOFA produces a binary file called **runSofa**, which can be found in the folder `%(SOFA_BUILD_DIR)/bin`. The execution of binary - either via the terminal or by double clicking the executable - launches a default XML scene with the name  `caduceus.scn`, using the Qt library.

The binary **runSofa** can be launched with different options, that can be enabled through the command line. You can display the different options available by using the argument `-h` in the command line: `./runSofa -h`.

* -a: **runSofa** starts the animation directly (no need to press on the animate button)
* -c: Displays interesting statistics about the computation time of the simulation.
* -g: allows you to choose between the existing user interfaces developed in SOFA.
* -l: allows you to load a SOFA plugin by specifying its name.
* \-n: specifies the number of simulation steps to run before closing **runSofa**. Can only be used with **-g batch**.

The default scene loaded by **runSofa** is named `"caduceus.scn"`. This scene file can be found in `%(SOFA_SOURCE_DIR)/examples/Demos`, along with other demo scenes.

### Create your scene in XML

#### XML Scene

Now let's take a look at the scene file. Scene files are XML files that describes the scene graph for the simulation. By convention, scene files have extension `".scn"`. You can find example scenes in the folder have mentioned below.

As you can see, the content of a scene file is written in XML. To create the simulation tree, you need to create a scene graph using the XML format. XML is a very simple language that allows the hierarchical description of a graph. Each element contained between chevrons are called "Tags". There are 2 types of tags: Nodes, and Leaves.

In terms of simulation, the **Node** tag is used to create a hierarchical level of modeling (e.g behavior model, collision model, visual model etc.). A scene file **always** contains a root node, that encapsulates your whole graph, and defines some overall specificities for your simulation (e.g gravity, dt, ...):

```xml
<Node name="root" gravity="0 -1000 0" dt="0.04">
    <Node name="Snake">
        <!-- some XML code -->
    </Node>
    <Node name="Collis">
        <!-- some collision-specific code -->
    </Node>
    <Node name="VisuBody" tags="Visual" >
        <!-- some rendering-specific code -->
    </Node>
    <!-- some more code... -->
    <Node name="Base">
        <!-- some more code... -->
    </Node>
</Node>
```

Nodes can be **nested**, as it is the case with every nodes inside the root node, allowing you to separate the different parts of your scenes. Nodes by themselves though, are useless, unless you combine them with some of the large collection of components available in SOFA. Those components are `"leaves"`, meaning that they cannot be nested. Leaves, **must** be contained inside the root node, or any other nodes in the scene graph.

#### XML properties: SOFA's data and links

As you can see in the scene file, nodes and components can have properties. In SOFA, xml properties are called Data, and allows you to access, and change some properties of the component it belongs to. This way for instance, a node can be activated or deactivated by setting its boolean data field **activated** to "true" or "false". Those data are the same parameters that you can find and modify in the "preperty" sheet of runSofa's interface.

```xml
<Node name="Snake" activated="true" >
    <!-- some code -->
</Node>
```

Moreover, dependency between Data containers of same nature can be specified in the XML scene files, indicating that the content of one container should be copied from the other, making the scene description fairly simple and above all efficient. To create this dependency, the flag "@" follow by the name of the source component should be used, as in following example:

```xml
<Node name="myNode">
    <meshobjloader name="myLoader" filename="mesh.obj">
    <mesh name="myMesh" src="@myLoader">
</Node>
```

The example above links the whole component `"myLoader"` to the property `"src"` of the component `"myMesh"`. But it is also possible to link more specifically the data of a component to the data of another one, gain, as long as those data are the same type:

```xml
<Node name="root" gravity="0 -1000 0" dt="0.04">
    <Node name="Loader">
        <meshvtkloader name="vtkLoader" filename="liver">
    </Node>
    <Node name="MechanicalModel">
        <mechanicalobject name="liverMO" scale="1" position="@../Loader/vtkLoader.position">
    </Node>
</Node>
```

The example above also shows the possibility to link to a component located in an other node. Here, the component vtkLoader is not located in the same node as the mechanical object component. By using `"../Loader/"` before component's name we want to link with, we are going up from one node, search for the `"Loader"` node, look for the component `"vtkLoader"` in it, and finally link with its data labeled `"position"`.

> ​	**Important Note**: It is not possible to link to a component that has not been yet declared in the scene graph. In other words, a component in the XML file only knows about the components declared earlier in the file.

#### Split your scene graph into multiple files

It is possible to include other xml files inside your scn files. This allows you to fragment your code to get a clearer and cleaner view of your scene. This can be achieved using the tag. For instance, the `"caduceus.scn"` could look like this:

```xml
<Node name="root" gravity="0 -1000 0" dt="0.04">
    <visualstyle displayFlags="showVisual  "> <!--showBehaviorModels showCollisionModels-->
    <lcpconstraintsolver tolerance="1e-3" initial_guess="false" build_lcp="false"  printLog="0" mu="0.2">
    <freemotionanimationloop>
    <collisionpipeline depth="15" verbose="0" draw="0">
    <bruteforcedetection name="N2">
    <minproximityintersection name="Proximity" alarmDistance="1.5" contactDistance="1">
    <lightmanager>
    <spotlight name="light1" color="1 1 1" position="0 80 25" direction="0 -1 -0.8" cutoff="30" exponent="1">
    <spotlight name="light2" color="1 1 1" position="0 40 100" direction="0 0 -1" cutoff="30" exponent="1">
    <collisionresponse name="Response" response="FrictionContact">

    <Node name="Snake">
        <include href="snake.scn">
    </Node>

    <Node name="Base" >
        <include href="base.scn">
    </Node>
</Node>
```

You would have the rest of the scene graph declared in two other files:

* `snake.scn`, containing the snake's graph
* `base.scn`, containing the pod's graph

### Monitor Component 

A Sofa Component named monitor (**`sofa::component::misc::Monitor`**) can help you to visualize, to monitor or to export some properties.

With this component, you can see the positions, trajectories, velocities, forces of chosen particles directly in the GUI or save it into files (readable with Gnuplot for example). You can get an idea of what this component can make launching the `Sofa/examples/Components/misc/Monitor.scn` scene.

### ExtraMonitor Component

The ExtraMonitor component gives you the ability to use everything that it is in Monitor with some Extra stuff. It has been written to allow to compute, for example, the resultant of the forces of all the dofs of a `MechanicalObject`, or the minimum displacement of a region...

### SOFA Mouse Manager

A Mouse manager has been created in Sofa, allowing to easily create and change the interactions with the different buttons of the Mouse. Note that the Shift key needs to be hold during the operation.

### SOFA launcher

The tool *sofa-launcher* case the scripting of **numerous SOFA simulations**. This can be done from XML or python scripts. To accelerate the processing of the simulations the script has the ability to run the simulation either: sequentially, in parallel or on a cluster.

There are two options to use it depending on your needs:

* You want to run a lot of simulation from you own python script.
* You don't want to write your own python script but still want to start a lot of simulations. You should have a look at the `sofa-launcher.py`application.



