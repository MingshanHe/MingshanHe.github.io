---
title: Rapidly-exploring_Random_Tree
date: 2022-04-05 22:15:17
tags: rrt algorithm
categories: Robotics
math: true
---
# Rapidly-exploring Random Tree (RRT)

  A **rapidly exploring random tree** (RRT) is an algorithm designed to efficiently search nonconvex, high-dimensional spaces by randomly building a space-filling tree. The tree is constructed incrementally from samples drawn randomly from the search space and is inherently biased to grow towards large unsearched areas of the problem.

  RRTs can be viewed as a technique to generate open-loop trajectories for nonlinear systems with state constraints. An RRT can also be considered as a Monte-Carlo Method to bias search into the largest Voronoi regions of a graph in a configuration space. Some variations can even be considered stochastic fractals.

  RRTs can be used to compute approximate control policies to control high dimensional nonlinear systems with state and action constraints.

## 1. Description

  Here is a brief description of the method for a general configuration space, $\C$ (this can be considered as a general state space that might include position and velocity information). An RRT that is rooted at a configuration $q_{init}$ and has $K$ vertices is constructed using the following:

```mathematica
Algorithm BuildRRT
    Input: Initial configuration qinit, number of vertices in RRT K, incremental distance Δq
    Output: RRT graph G

    G.init(qinit)
    for k = 1 to K do
        qrand ← RAND_CONF()
        qnear ← NEAREST_VERTEX(qrand, G)
        qnew ← NEW_CONF(qnear, qrand, Δq)
        G.add_vertex(qnew)
        G.add_edge(qnear, qnew)
    return G
```

  Step 3 chooses a random configuration, $q_{rand}$, in $\C$. Alternatively, one could replace *RAND_CONF* with *RAND_FREE_CONF*, and sample configurations in $\C_{free}$ (by using a collision detection algorithm to reject samples in $\C_{obs}$).

  It is assumed that a metric $\rho$ has been defined over $\C$. Step 4 selects the vertex, $x_{near}$ in the RRT, $G$, that is closest to $q_{rand}$. 

  In Step 5 of *BUILD_RRT*, *NEW_CONF* selects a new configuration, $q_{new}$ by moving from $q_{neaer}$ an incremental distance, $\Delta q$, in the direction of $q_{rand}$. This assumes that motion in any direction is possible. If differential constraints exist, then inputs are applied to the corresponding control system, and new configurations are obtained by numerical integration. Finally, a new vertex, $q_{new}$, and is added, and a new edge is added from $q_{near}$ to $q_{new}$.

  To gain a better understanding of RRTs, consider the special case in which $\C$ is a square region in the plane. Let $\rho$ represent the Euclidean metric. The frames below show the construction of an RRT for the case of $\C=[0,100]\times[0,100]$, $\Delta q = 1$, and $q_{init}=(50,50)$

<img src="/image/Robotics/Rapidly-exploring_Radom_Tree/1.gif" alt="2" style="zoom:80%;" />
## 2. Conclusion

  The RRT quickly expands in a few directions to quickly explore the four corners of the square. Although the construction method is simple, it is no easy task to find a method that yields such desirable behavior.

  Consider, for example, a rand random tree that is constructed incrementally by selecting a vertex at random, an input at random, and then applying the input to generate a new vertex. Although one might intuitively expect the tree to "randomly" explore the space, there is actually a very strong bias toward places already explored (simulation experiments yield an extremely high density of vertices near $q_{init}$, with litter other exploration). A random walk also suffers from a bias toward places already visited. An RRT works in the opposite manner by being biased toward places not yet visited.

  This can be seen by considering the Voronoi diagram of the RRT vertices. Larger Voronoi regions occur on the "frontier" of the tree. Since vertex selection is based on nearest neighbours, this implies that vertices with large Voronoi regions are more likely to be selected for expansion. On average, an RRT is constructed by iteratively breaking large Voronoi regions into smaller ones.