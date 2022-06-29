---
title: SOFA Study Record (2)
date: 2022-06-29 10:12:22
categories: Soft Robot
tags: [Soft Robot, Simulation]
math: true
---

## Scene Graph

A simulation in SOFA is described as a scene with an intrinsic generalized hierarchy. This scene is composed of nodes organized as a tree or as a Directed Acyclic Graph (DAG). The different simulated objects are described in separate nodes, and different representations of a same object can be done in different sub-nodes.

#### Structure of a scene

The scene starts from a parent node, called the `"Root"` node. All other nodes (called child nodes) inherit from this main node. This design is highly modular, since components in the scene are independent of each other. One physical model (springs with Spring Force Filed) could be simply replaced by another one (Triangular FEM with Triangle FEM Force Field) by changing one component in the graph. In the same way, an explicit integration scheme (Euler Solver) could be replaced by an implicit one (Euler Implicit Solver) by modifying one XML line in the scene file. This high modularity of the framework is induced by the scene graph-visitor approach described below.

Nodes can be structured serially in the graph. Such a hierarchical graph allows for having several representation of a same object. For example, the child node "Liver" implements the mechanical behavior of the liver, whereas the sub-node "Visual" describes a surface model of the liver.

To build a simulation in SOFA, the scene graph can be written both using:

* XML files.
* Python scripts.

### Data

Each of these components (solvers, forcefield, mass) contains attributes. For instance, a component of mass features an attribute fro mass density; an iterative linear solver needs an attribute defining a maximum of iterations. These attributes are also called **Data**. These data are containers providing a reflective API used for serialization in XML files and automatic creation of input/output widgets in the user interface.

Two data instances can be connected one with another to keep their value synchronized. This is only possible if they both have the same type (`float` , `vector<double>`). A mechanism of lazy evaluation is used to recursively flag data that are not up-to-date. Then, the data is recomputed (only if necessary). The network of interconnected Data objects defines a data dependency graph. In an XML file, one data is connected to another when "@" is used.

### Tags

Any component can be set with one or several `"Tags"`. The `"Tags"` data field is available for any SOFA component. A tag is useful to find a specific component in the scene, to distinguish several instances of a same class in the scene graph or to process these instances differently one from another.

## Animation Loop

All the scenes in SOFA must include an ***Animation Loop***. This components orders all steps of the simulation and the system resolution. At each time step, the animation loop triggers each event (solving the matrix system, managing the constraints, detecting the collision, etc.) through a Visitor mechanism (see below). In a scene, if no animation loop is defined, a `"DefaultAnimationLoop" `is automatically created.

There are several ***Animation Loop*** available in SOFA:

* ***`DefaultAnimationLoop`***: This is the default animation loop as the name indicates! This animation loop is included by default at the root node of the graph, if no animation loop is specified in the scene. With a *`DefaultAnimationLoop`*, the loop of one simulation step follows:
  * collision detection is triggered through the collision pipeline (if any)
  * solve the physics in the scene by triggering the integration scheme, taking the constraint, collision into account
  * update the system (new values of the dofs), the context (dt++), the mappings and the bounding box (volume covering all objects of the scene).
* ***`MultiTagAnimationLoop`***: This animation loop works by labeling components using different tags. With a *`MultiTagAnimationLoop`*, the loop of one simulation step is the same as the *`DefaultAnimationLoop`*, except that one tag is solved after another, given a list of tags:
  * For each tag defined: 1. collision detection is triggered through the collision pipeline 2. solve the physics in the scene by triggering the integration scheme, taking the constraint, collision into account
  * update the system (new values of the dofs), the context (dt++), the mappings and the bounding box (volume covering all objects of the scene).
* ***`MultiStepAnimationLoop`***: Given one time step, this animation loop allows for running several collision ($C$ being the number of collision steps) and several integration time in one step ($I$ being the number of integration time steps), where the two ($C$ and $I$) can be different. The loop in one animation step is:
  * compute $C$ times the collision pipeline within one time step $dt$
  * For each collision step, solve $I$ times the linear system for time integration using the time step $dt'$
  * update the context, the mappings, the bounding box (the visualization is done once at each time step $dt$)
* ***`FreeMotionAnimationLoop`***: this animation loop is used for simulation involving constraints and collisions. With a *`FreeAnimationLoop`*, the loop of one simulation step follows:
  * build and solve all linear system in the scene without constraints and save the "free" values of the dofs
  * collision detection is computed thus generating constraints
  * constraints are solved as one system to compute a correction term taking into account the collision & constraints
  * update the mappings, the bounding box

## Visitors

During the different steps of the simulation (initialization, system assembly, solving, visualization), information needs to be recovered from all graph nodes. SOFA relies on an implicit mechanism: the ***Visitor***. You can find the abstract *Visitor* class in the Sofa simulation package.

*Visitors* traverse the scene top-down and bottom-up and call the corresponding virtual functions at each graph node traversal. *Visitors* are therefore used to trigger actions by calling the associated virtual functions. Algorithmic operations on the simulated objects are implemented by deriving the *Visitor* class and overloading its virtual functions *`processNodeTopDown()`* and *`processNodeBottomUp()`*.

This approach hides the scene structure (parent, children) from the components, for more implementation flexibility and a better control of the execution model. Moreover, various parallelism strategies can be applied independently of the mechanical computations performed at each node. The data structure is actually extended from strict hierarchies to directed acyclic graphs to handle more general kinematic dependencies. The top-down node traversals are pruned unless all the parents of the current node have been traversed already, so that nodes with multiple parents are traversed only once all their parents have been traversed. The bottom-up traversals are made in the reverse order.



