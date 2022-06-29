---
title: Dynamic-Q
date: 2022-06-27 21:40:42
tags: [Reinforcement Learning, Algorithm]
math: true
---

## Introduction

In reinforcement learning, "model" usually refers to the model of the environment with which the intelligent agent interacts. And that is modeling the state transition probability and reward function of the environment. There are two types of reinforcement learning algorithms according to whether there is an environment model: **model-based** and **model-free** reinforcement learning.

Model-free reinforcement learning directly performs policy improvement or value estimation based on the data sampled by the agent and the environment.

The **Dynamic-Q** algorithm is also a very basic model-based reinforcement learning algorithm, but its environment model is estimated from sampled data.

Reinforcement learning algorithms have two important evaluation indicators: one is the expected return of the algorithm in the initial state after the algorithm converges, and the other is the sample complexity, which is the number of samples that the algorithm needs to sample in the real environment to achieve the convergence result. Since **model-based** reinforcement learning algorithms have an environment model, the agent can additionally interact with the environment model, and the demand for samples in the real environment is often reduced, so it usually has lower samples than model-free reinforcement learning algorithms exactly for the complexity. However, the environment model may not be accurate enough to completely replace the real environment, so the expected return of its policy after the model-based reinforcement learning algorithm converges may not be as good as that of the model-free reinforcement learning algorithm.

## Details

The Dynamic-Q algorithm is a classic **model-based** reinforcement learning algorithm. It uses a method called Q-planning to generate some simulated data based on the model, and then use the simulated data along with the real data to improve the policy. Q-planning selects a previously visited state at a time ***s***, take an action that was once performed in that state ***a***, the state after the transition is obtained through the model ***s\'***, and reward ***r***, and according to this simulated data ***(s, a, r, s\')***, use the Q-learning update method to update the action value function.

Let's take a look at the specific process of the Dyna-Q algorithm:

* Initialization ***Q(s,a)***, to initialize the model ***M(s,a)***
  * for sequence = 1 to ***E*** do:
    * get the initial state ***s***
    * for t = 1 to ***T*** do:
      * use $\epsilon$ - greedy strategy based on ***Q*** select current state ***s*** the next action ***a***
      * get feedback from the environment ***r***, ***s\'***
      * $Q(s,a) \leftarrow Q(s,a) + \alpha[r + \gamma max_{a'}(Q(s',a')-Q(s,a))]$ 
      * $M(s,a)\leftarrow r, s'$
      * for times ***n*** = 1 to ***N*** do:
        * Randomly choose a state that has been visited $s_m$
        * tack an action $a_m$ previous acted in state $s_m$ 
        * $r_m, s'_m \leftarrow M(s_m, a_m)$
        * $Q(s_m,a_m)\leftarrow Q(s_m,a_m) + \alpha[r_m + \gamma max_{a'}(Q(s'_m,a')-Q(s_m,a_m))]$
      * end for
      * $s\leftarrow s'$
    * end for
  * end for
  
  And you can see that after each Q-learning interaction with the environment , Dynamic-Q will do ***n*** Sub-Q-Planning. And the number of Q-planning ***N*** is a hyper-parameter that can be selected in advance, and when it is 0, the algorithm is normal Q-learning.
